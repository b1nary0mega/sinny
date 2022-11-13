# Enumeration

Current IP: 10.129.173.59

## nmaps

```
PORT    STATE SERVICE REASON
873/tcp open  rsync   syn-ack ttl 63

sudo nc -vn 10.129.173.59 873
10.129.173.59 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
public          Anonymous Share
@RSYNCD: EXIT
```

Let's sync the folder locally

```
mkdir public && sudo rsync -av rsync://10.129.173.59/public ./public
receiving incremental file list
./
flag.txt

sent 50 bytes  received 161 bytes  24.82 bytes/sec
total size is 33  speedup is 0.16
```
