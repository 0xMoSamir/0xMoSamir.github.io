---
title: "Challenge Name: ConCmarks"
date: 2024-11-26 08:45:00 +0000
categories: [CTF, CyberTalents]
tags: [Writeup, Cybersecurity]
pin: true
---
it might be useful to find a mark.

this is our target:
![Screenshot](/assets/img/concmarks/image.png)

Okay guys, I guess this challenge It depends on exploring something mysterious, maybe a comment or something in the page source or something I will figure it out when I inspect the page, so let’s open the page source:
![Screenshot](/assets/img/concmarks/image1.png)

Yea as we can see, there is nothing interesting here just very basic HTML Codes.
let’s inspect the main page and see :
![Screenshot](/assets/img/concmarks/image3.png)

yea we got this pretty comment, let’s explain it :
FILE= sourceXXXX : Suggests a file or resource named "sourceXXXX," where "XXXX" is likely the part I need to figure out.
XXXX are numbers > 7000 & < 9000 : Indicates that "XXXX" is a number between 7001 and 8999.

So I need a tool to try this on the target so, I’ll use burpsuite the intruder model, I’ll edit the url and add sourceXXXX and I will replace XXXX with a different number starts from 7000 until 8999 and I will make a cup of tea until it will finish it’s job, so let’s do it:
this is the url before adding the file source in the directory:

http://wcamxwl32pue3e6mekgvd1gf9zrqqyz8wqrwf9vw-web.cybertalentslabs.com/

I will add sourceXXXX

http://wcamxwl32pue3e6mekgvd1gf9zrqqyz8wqrwf9vw-web.cybertalentslabs.com/sourceXXXX

![Screenshot](/assets/img/concmarks/4.png)

As u see here I edited XXXX and I added it between §§ because it’s the part that I will change in every request, so let’s create our payload:

Oookkayy actually burpsuite will take too much time to do that, so I’ll use a python script to do this task for me to just save time..

import requests
import time

url = "http://wcamxwl32pue3e6mekgvd1gf9zrqqyz8wqrwf9vw-web.cybertalentslabs.com/source"

for number in range(7001, 9000):  # Iterate through the range 7001 to 8999
    full_url = f"{url}{number}"  # Construct the full URL
    try:
        response = requests.get(full_url)  # Send a GET request
        if response.status_code == 200:  # Check for successful response
            print(f"Valid file found: source{number}")
        else:
            print(f"Checked: source{number} - Status Code: {response.status_code}")
        time.sleep(0.2)  # Add a small delay to avoid rate-limiting issues
    except requests.exceptions.RequestException as e:  # Handle any errors
        print(f"Error with {full_url}: {e}")
    
    Imports:
-requests: For sending HTTP requests.
-time: For adding delays between requests.

Base URL:
url: The starting URL provided in the challenge.

Loop:
for number in range(7001, 9000): Iterates through numbers from 7001 to 8999.

Full URL:
full_url = f"{url}{number}": Constructs the full URL by appending the number.

HTTP Request:
response = requests.get(full_url): Sends a GET request to the constructed URL.

Check Status:
If response.status_code == 200, it prints the valid file name.
Else, it logs the status code for debugging.

Delay:
time.sleep(0.2): Pauses 0.2 seconds between requests to avoid rate-limiting.

Error Handling:
Catches and logs any exceptions during the requests.
this is the script that I will use, I will save it as a check.py file
![Screenshot](/assets/img/concmarks/5.png)

then, I will run the code from my terminal using this command
"python check.py"
![Screenshot](/assets/img/concmarks/6.png)

it’s working but still without 200 OK response..
![Screenshot](/assets/img/concmarks/7.png)

and yeaaa finally we got the right source :
let’s add it into the url and see the result…
![Screenshot](/assets/img/concmarks/8.png)

and we have been redirected to this page, let’s explain this code..

The PHP code checks two inputs, n1 and n2, ensuring they are different but produce the same MD5 hash when concatenated with a $salt. To solve this challenge, we need to find two distinct values for n1 and n2 that cause an MD5 hash collision with the given $salt.

let’s try to add a random value into n1&n2 and see:
![Screenshot](/assets/img/concmarks/9.png)

as we can see:

Sorry this value not valid.
So I will make a quick research about how could I bypass this condition.

this condition treats the inputs as a string but what if we let it treat it as an array?

so in this case we could bypass it, to be more clear we will bypass it logically :

n1[] as an Array:
By passing n1[]=, $_GET['n1'] becomes an array instead of a string.
n2[]=1:

Similarly, $_GET['n2'] becomes an array.
Bypass Logic:

The !== operator compares input1 (array) with input2 (array) and evaluates them as not strictly equal, even if their content matches.
Hashing an array with @hash("md5", $salt.$input1) results in NULL, which is equal for both n1[] and n2[] due to the silent error suppression (@).
let’s try it and see the result, our payload will be like
[text](http://wcamxwl32pue3e6mekgvd1gf9zrqqyz8wqrwf9vw-web.cybertalentslabs.com/?n1[]=&n2[]=1)

and yea we got the flag!
![Screenshot](/assets/img/concmarks/10.png)

FLAG{K0nC473n4710N_!5_50_C00l}

thanks for reading my writeup!
See u in the next CTF Challenge…