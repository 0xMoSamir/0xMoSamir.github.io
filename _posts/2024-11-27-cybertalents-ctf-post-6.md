---
title: "Challenge Name: catch me if you can"
date: 2024-11-27 12:00:00 +0000
categories: [CTF, CyberTalents]
tags: [Writeup, Cybersecurity, CyberTalents]
pin: false
---

I'm Just wanna Make Sure if you Are Mr.Robot

**this is our target:**

![Screenshot](/assets/img/catchmeifucan/1.png)

let's start by inspecting the home page and the other pages in nav bars..

found nothing intersting at all, let's brute force the directories to discover any important path.

I’ll Use *ffuf* (Fuzz Faster U Fool):

`ffuf -u http://wcamxwl32pue3e6mxmqvrwdh2zokqyz8wqrwf9vw-web.cybertalentslabs.com//FUZZ -w /path/to/wordlist.txt`

*So We ‘ve got this Response:*

![Screenshot](/assets/img/catchmeifucan/2.png)

```c 
robots.txt
index.php 
flag.php 
source.php
```

So, let’s use every single path in our target url and see the result:

![Screenshot](/assets/img/catchmeifucan/3.png)

```c
User-agent: *
Disallow: /S3cr3t.php
Disallow: /source.php
```

So, Let’s Try → `S3cr3t.php`

![Screenshot](/assets/img/catchmeifucan/4.png)

hmmm, Intersting but let’s check → `source.php` before we continue..

![Screenshot](/assets/img/catchmeifucan/5.png)

We ‘ve got these codes:

```php
<?php

include(‘flag.php’);

$password=$_POST[‘pass’];

if (strpos( $password, ‘R_4r3@’)!== FALSE){

if (!preg_match(‘/^-?[a-z0–9]+$/m’, $password)) {

die(‘ILLEGAL CHARACTERS’);

}
echo $cipher;
}
else
{
echo ‘Wrong Password’;
}

?>
```


**Summary:**

The script checks if the submitted password contains the substring `'R_4r3@'`.

If it does, it checks that the password contains only lowercase letters, numbers, and an optional hyphen at the start.

If both conditions are met, it outputs a secret value (stored in `$cipher`).

If the password fails any of these checks, it either displays `Wrong Password` or `ILLEGAL CHARACTERS` based on the failure.

Let’s Try to login with `'R_4r3@'` and see what’s gonna happen:

![Screenshot](/assets/img/catchmeifucan/6.png)


**why it gave us this response ‘ILLEGAL CHARACTERS’ ?**

`'R_4r3@'` contains characters that will fail the regular expression check (because `R`, `_`, and `@` are not allowed).

**Understanding the Regex /^-?[a-z0-9]+$/m:**

The password must contain only lowercase letters and numbers.
 So, it cannot include the substring `'R_4r3@'` without triggering the `ILLEGAL CHARACTERS` response.

**`strpos()` Function:**

`strpos()` will only check if the substring exists in the password. However, it does not require the password to only contain `'R_4r3@'`, so you might be able to add this substring in a way that passes both checks.

So Now We Need to bypass this Restricted Area, So I’m gonna give the javascript code that was given in the source.php path and I’m gonna ask him to how to bypass it and I ‘ve got this payload : `test%0AR_4r3@`

So let’s try it by using `Curl` Tool:

`curl http://wcamxwl32pue3e6mxmqvrwdh2zokqyz8wqrwf9vw-web.cybertalentslabs.com/S3cr3t.php -d “pass=test%0AR_4r3@”`

![Screenshot](/assets/img/catchmeifucan/7.png)

The `test%0AR_4r3@` password adds a newline `(%0A)` between `test` and `R_4r3@`.

This exploits the logic of the script, making the `strpos()` function match `'R_4r3@'`, while allowing the rest of the password `(test)` to pass the regex check.

So we found something here will lead us to the flag!


Can You Read This For Me ? `<br><br>-[ — — — ->+<]> — -.++++++. — — — — — — . — [ — ->+<]> — -.[ — — ->+<]>.[ — ->++<]>.>-[ — — ->+<]>.>-[ — ->+<]> — .[ — ->+<]>+++. — . — [->+++++<]>+. — -[ →+++<]> — .+[ — — ->+<]>.++[++> — -<]>.+[->++<]>. — — -.+[ — ->++<]>+. — [ — — ->+<]>-.>-[ — — ->+<]>.+.> — [ →+++<]>.`


It’s Called `Brainf*ck` Code, So let’s find some tool to decode it.

and this is our flag: FL@g{R3Str1Ct1d_Ar34}





