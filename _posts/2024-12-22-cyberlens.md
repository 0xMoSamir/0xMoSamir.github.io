---
title: "CyberLens"
date: 2024-12-1 12:00:00 +0000
categories: [CTF, Tryhackme]
tags: [Writeup, Cybersecurity, Tryhackme]
pin: flase
---

Can you exploit the CyberLens web server and discover the hidden flags?

Let's see If we can do this guys, Do you think so?

![Scennshot](/assets/img/CyberLens/image.png)

Before we start let's make sure that we added the domain in the hosts file.

`Things to Note`
`1. Be sure to add the IP to your /etc/hosts file:`

```bash
sudo echo '10.10.20.65 cyberlens.thm' >> /etc/hosts
```

`2. Make sure you wait 5 minutes before starting so the VM fully starts each service`

So Let's make a nmao to scan the open ports on the system:

```bash
nmap -p- cyberlens.thm -T4
```

![Scennshot](/assets/img/CyberLens/1.png)


