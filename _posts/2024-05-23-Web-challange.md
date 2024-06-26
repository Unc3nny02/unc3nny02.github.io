---
layout: post
title:  "ApacheBlaze"
---

## Hello Everyone
Today we be doing ApacheBlaze.

![Screenshot 2024-06-14 151309](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/4dd73c9b-c4b3-4d97-a92f-9fd471844343)

ApacheBlaze is a HackTheBox challenge categorized under web tasks. It's considered moderately straightforward in difficulty, making it a good opportunity to understand Apache proxy mechanics and explore manipulating HTTP request headers.

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/08910647-af2d-476a-83a3-4f298418931d)


Understanding the purpose of the website

Nothing special to understand here. The website allows us to choose four diffirent kind of games. To further understand the structure of the website lets analyze the source code.

Understanding what we want to access

After downloading and looking at the files of the website, we notice important information :

Apache2 is used with the mod_proxy module

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/27a95643-ac59-44b3-b685-42793b00ad46)

And we can see there’s 2 backend virtual hosts with a reverse proxy and a kind of load balancer

Call the game API with click_topia arg and the X-Forwarded-Host equal to dev.apacheblaze.local will return the flag

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/18585545-209c-4e3a-8001-cf9387a9c6ef)

The Solution

Before starting to exploit, it’s important to understand how Apache and mod_proxy module work : Apache documentation.

The first thing we want to try is to request the API and setting the X-Forwarded-Host header to dev.apacheblaze.local, indeed, that looks very simple. Let’s try !

We try to navigate to <website-ip><port>/api/games/click_topia, and we see 

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/5dd5d319-1b85-44cb-8965-f0cf2dced102)

But why ? Let’s run the website locally and print the headers 

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/538cb82d-395e-44e5-9a8e-8c3ae2a34d8c)

Here we can see that the X-Forwarded-Host contains dev.apacheblaze.local but also 2 other elements. In the Apache documentation, we can understand why 


```shell
When acting in a reverse-proxy mode (using the ProxyPass directive, for example), mod_proxy_http adds several request headers in order to pass information to the origin server. These headers are:

X-Forwarded-For
    The IP address of the client.
X-Forwarded-Host
    The original host requested by the client in the Host HTTP request header.
X-Forwarded-Server
    The hostname of the proxy server. 
```

Now that’s clear, the 2 virtual hosts (reverse proxy and load balancer) on port 1337 and 8080 are appended to the X-Forwarded-Host header during the request treatment.

So we’ll have to find an exploit/bypass to set this header properly, with only the wanted value. With all the information we previously found, we can find an exploit : HTTP Request Smuggling Attack (CVE-2023–25690). Here is a github repo explaining how to use the exploit CVE-2023–25690.

Now let’s cook !

With this exploit, we learn that we can hide a second request in the first we sent to the target (with the \r\n\r splitting method). That enable us to send the request directly from the reverse proxy, so we will not have other stuff appended to the end of X-Forwarded-Host header.

Here are the two requests I’m gonna send in one, the second is just here to permit us to send the first one from the reverse proxy, you can put any route you want 

```shell

GET /api/games/click_topia HTTP/1.1
Host: dev.apacheblaze.local


GET / HTTP/1.1
Host: localhost:1337

-----------------------------------

Reminder for encoding : 

\r\n     ->  %0d%0a
\r\n\r   ->  %0d%0a%0d

```
and i send it through burpsuite

![image](https://github.com/Unc3nny02/unc3nny02.github.io/assets/127601349/96bd6191-2758-48b7-9fad-9ea004b61e7b)

On both ports, I successfully retrieved the flag. Additionally, it’s worth noting that I could access localhost:8080 by specifying it as either (127.0.0.1:8080 or 0.0.0.0:1337).

Thank You for reading my blog hope you learn something new

