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


<img width="955" alt="ustawienia port 22" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/20c185ff-4c61-4e15-ab1f-ce449cbbc802">

After refreshing page eeverything should work's just fine.

In CTF's when we're dealing with an http server, it's always good to check page source.
Bingo! We found a comment encoded with base64:

<img width="957" alt="base" src="https://github.com/Th3l3mic/WriteUps/assets/167564930/f764308f-4524-43d2-8208-96a046ae46e5">
After decoding it's states:

<code>Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq</code>

We have na some password and John Graves could be a clue, but we're gonna leave it for now. Let continue inspecting the page. As we can see there's three photos on the page and end point <code>/recovery.php</code>

The photos could be nice finding's, one of this photos is even called stego.jpg, that implies's that we're gonna probably dealing with steganography.
