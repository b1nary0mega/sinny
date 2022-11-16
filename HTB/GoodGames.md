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
