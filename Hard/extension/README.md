
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

Adding those subdomains into my `/etc/hosts` file, it's time for more endpoint enumeration

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


