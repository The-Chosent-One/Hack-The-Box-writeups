
![image](https://user-images.githubusercontent.com/57739806/218311539-fb920584-7f00-4e6b-813a-00dbf09d7892.png)
> This is in the Hard category of boxes in HTB

# Enumeration
We start off with some standard enumeration, with nmap

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
