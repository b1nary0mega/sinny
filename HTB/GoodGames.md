#HTB - Good Games
Keywords: #python #docker #weak-pwd

## Details

_Current IP:_ 10.129.15.202

_Host Name:_ goodgames.htb  (added to /etc/hosts)

### nmaps

```
$ sudo nmap -sS -sC -vv -p- -T4 -Pn -oA nmap-T4-all_tcp 10.129.15.202
Nmap scan report for 10.129.15.202
Host is up, received user-set (0.11s latency).
Scanned at 2022-11-16 13:17:48 EST for 226s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-favicon: Unknown favicon MD5: 61352127DC66484D3736CACCF50E7BEB
|_http-title: GoodGames | Community and Store

Read data files from: /usr/bin/../share/nmap
# Nmap done at Wed Nov 16 13:21:34 2022 -- 1 IP address (1 host up) scanned in 227.19 seconds
```

```
$ sudo nmap -sS -p 80 -sV 10.129.15.202              
[sudo] password for jimmi: 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-16 13:34 EST
Nmap scan report for 10.129.15.202
Host is up (0.11s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
Service Info: Host: goodgames.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.04 seconds
```
### Versions

```
Apache httpd 2.4.51
JavaScript frameworks - GSAP 1.20.3
Web frameworks - Flask 2.0.2
Web servers - Flask 2.0.2
Programming languages - Python 3.9.2
JavaScript libraries:
- Hammer.js 2.0.7
- jQuery 3.3.1
- Moment.js 2.22.1
```

### gobusting

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://goodgames.htb -s '200,204,301,302,307,403,500' -b '' --exclude-length 9265 -e | tee gobuster-common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:              http://goodgames.htb
[+] Method:           GET
[+] Threads:          10
[+] Wordlist:         /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:     302,307,403,500,200,204,301
[+] Exclude Length:   9265
[+] User Agent:       gobuster/3.3
[+] Expanded:         true
[+] Timeout:          10s
===============================================================
2022/11/16 14:19:03 Starting gobuster in directory enumeration mode
===============================================================
http://goodgames.htb/blog                 (Status: 200) [Size: 44212]
http://goodgames.htb/forgot-password      (Status: 200) [Size: 32744]
http://goodgames.htb/login                (Status: 200) [Size: 9294]
http://goodgames.htb/logout               (Status: 302) [Size: 208] [--> http://goodgames.htb/]
http://goodgames.htb/profile              (Status: 200) [Size: 9267]
http://goodgames.htb/server-status        (Status: 403) [Size: 278]
http://goodgames.htb/signup               (Status: 200) [Size: 33387]
===============================================================
2022/11/16 14:20:03 Finished
===============================================================

$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/CGIs.txt -u http://goodgames.htb -s '200,204,301,307,403,500' -b '' --exclude-length 9265 --exclude-length 85107 -e | tee gobuster-cgis.txt                                                                                                                 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:              http://goodgames.htb
[+] Method:           GET
[+] Threads:          10
[+] Wordlist:         /usr/share/seclists/Discovery/Web-Content/CGIs.txt
[+] Status codes:     301,307,403,500,200,204
[+] Exclude Length:   9265,85107
[+] User Agent:       gobuster/3.3
[+] Expanded:         true
[+] Timeout:          10s
===============================================================
2022/11/16 14:31:07 Starting gobuster in directory enumeration mode
===============================================================
Progress: 168 / 3389 (4.96%)[ERROR] 2022/11/16 14:31:10 [!] parse "http://goodgames.htb/%NETHOOD%/": invalid URL escape "%NE"
Progress: 446 / 3389 (13.16%)[ERROR] 2022/11/16 14:31:13 [!] parse "http://goodgames.htb/%a%s%p%d": invalid URL escape "%a%"
Progress: 489 / 3389 (14.43%)[ERROR] 2022/11/16 14:31:14 [!] parse "http://goodgames.htb/default.htm%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%": invalid URL escape "%"
http://goodgames.htb/server-status        (Status: 403) [Size: 278]
===============================================================
2022/11/16 14:31:50 Finished
===============================================================

$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://goodgames.htb -s '200,204,301,302,307,403,500' -b '' --exclude-length 9265 -e -x py,js,txt | tee gobuster-dirlist_med.txt 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:              http://goodgames.htb
[+] Method:           GET
[+] Threads:          10
[+] Wordlist:         /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Status codes:     403,500,200,204,301,302,307
[+] Exclude Length:   9265
[+] User Agent:       gobuster/3.3
[+] Extensions:       js,txt,py
[+] Expanded:         true
[+] Timeout:          10s
===============================================================
2022/11/16 14:49:04 Starting gobuster in directory enumeration mode
===============================================================
http://goodgames.htb/blog                 (Status: 200) [Size: 44212]
http://goodgames.htb/login                (Status: 200) [Size: 9294]
http://goodgames.htb/profile              (Status: 200) [Size: 9267]
http://goodgames.htb/signup               (Status: 200) [Size: 33387]
http://goodgames.htb/logout               (Status: 302) [Size: 208] [--> http://goodgames.htb/]
Progress: 19750 / 830576 (2.37%)
```
