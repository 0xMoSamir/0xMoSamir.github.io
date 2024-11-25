---
title: "Challenge Name:Secret Browser"
date: 2024-11-25 12:00:00 +0000
categories: [CTF, CyberTalents]
tags: [Writeup, Cybersecurity, CyberTalents]
pin: true
---
The company employees is using company special browser to view the website content.

This is our target:

![Screenshot](/assets/img/secret%20browser/image.png)

so, let’s see the source page to see if there anything important:
![Screenshot](/assets/img/secret%20browser/image-1.png)

just a simple message: Welcome Guest , your are not using our company browser.
so let’s open our burpsuite to get more information about this page:
I intercepted the page request and I focused on the User-Agent line because it’s the condition that restricts us from accessing the page as users so in this case it was Mozilla fire fox and we got the message Welcome Guest , your are not using our company browser
![Screenshot](/assets/img/secret%20browser/image-2.png)

So, Now we need to know the Company name and edit it the User-Agent line in the request then the new browser name will be PublicTradeCo
when I saw the source page I got the Company name in the title line in HTML File:
![Screenshot](/assets/img/secret%20browser/image-3.png)

PublicTradeCo company for trading
So, I will replace Mozilla with PublicTradeCo to see is it gonna work or not..
![Screenshot](/assets/img/secret%20browser/image-4.png)

and yea after changing the User-Agent browser we got a different response message now which is: Welcome employee , the flag you are looking for is here somewhere
flag is in the response header:
![Screenshot](/assets/img/secret%20browser/image-5.png)

So yea this is our Flag : W3lcomeC0mpanyUs3R

Wish to see you in the next challenge bro 