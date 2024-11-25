---
title: "CyberTalents CTF Challenge Writeup"
date: 2024-11-25 12:00:00 +0000
categories: [CTF, CyberTalents]
tags: [Writeup, Cybersecurity, CyberTalents]
pin: true
---

Challenge Name: admin gate first
Flag is safe in the admin account info

In this post, I'll walk you through my approach to solving the "Admin Gate First" from the CyberTalents CTF.
![Screenshot](/assets/img/image.png)

By analyzing the challenge name, we can deduce what is required of us in the challenge. For instance, by examining both the name and description, we can identify that it involves a Broken Access Control vulnerability. This means we will exploit the vulnerability to gain control over the admin account. But first, let’s revisit the concept of Broken Access Control.

Broken Access Control is a security vulnerability that occurs when an application fails to properly enforce user permissions. This oversight allows unauthorized users to access resources, functions, or data that should be restricted. It is a common flaw in web applications and a critical security issue, as it can result in data breaches, privilege escalation, and unauthorized control of systems.

Let’s get back to our challenge:
![Screenshot](/assets/img/image-1.png)


The interface we are analyzing contains two input fields. At the top of the page, it provides us with a set of credentials:
User: test
Pass: test
Let’s use the provided credentials and press "Login" to proceed.
![Screenshot](/assets/img/image-2.png)

I’ve solved this challenge on TryHackMe before, and its main concept revolves around modifying the username and role. However, it’s not as straightforward as it sounds—both the username and role are encrypted. To proceed, we’ll open the challenge in Firefox and use the Developer Tools, which will give us better control and insight into the encrypted data.
![Screenshot](/assets/img/image-3.png)
and after that we will open Storage section and look at the Cookies and we will see :
![Screenshot](/assets/img/image-4.png)

in the cookies we found a JWT token and it contains three parts, every dot separates each part and to make it easy for you I used 3 diff colors to highlight each part, now We need to decode each part, and I will use my kali terminal to decode it using the following command:
![Screenshot](/assets/img/image-5.png)
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcInRlc3RcIixcInJvbGVcIjpcInVzZXJcIn0ifQ.XSPy0jZd8CEtHl2e3C1SjPaewco1tjO3iajbkJy2OFQ

echo ”our encoded texts” | base64 -d
![Screenshot](/assets/img/image-6.png)

the header part of the decoding gave us :
{“typ”:”JWT”,”alg”:”HS256"}
means:
The token is a JWT.
It uses HMAC-SHA256 as the signing algorithm.
so let’s decode the payload part :
![Screenshot](/assets/img/image-7.png)
The signature part is 123456 I found it by using jwt_tool
so we look like we are close to get the flag, all what we have to do rn is change the role from user to admin and refresh the page to see what is gonna happen.
I will edit the text first then I’ll encode it to base64 by using this command:
echo -n ‘your_string’ | base64
Edit the JWT Token
{
“typ”: “JWT”,
“alg”: “HS256”
}
{
“data”: “{\”username\”:\”test\”,\”role\”:\”admin\”}”
}

the new token:
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcInRlc3RcIixcInJvbGVcIjpcImFkbWluXCJ9In0.jHijo7pYELY4DJG20h2xCRiKRqW3HCdB1AnyOGsTNUw

and then we will put the new token in the value column and press on refresh the page and congratulations you found the flag:
![Screenshot](/assets/img/image-8.png)

![Screenshot](/assets/img/image-9.png)

The Flag is : J!W!T#S3cr3T@2018

Happy Hacking..!

