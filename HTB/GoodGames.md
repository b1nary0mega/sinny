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
```

### Wappalyzer

Photo galleries
- PhotoSwipe

JavaScript frameworks
- GSAP 1.20.3

Video players
- YouTube
- Twitch Player

Font scripts
- Font Awesome
- Ionicons
- Google Font API

Web frameworks
- Flask 2.0.2

Miscellaneous
- Popper

Web servers
- Flask 2.0.2
Programming languages
- Python 3.9.2

JavaScript libraries
- Hammer.js 2.0.7
- jQuery 3.3.1
- Moment.js 2.22.1
- Flickity
- PhotoSwipe
- SoundManager

UI frameworks
- Bootstrap
