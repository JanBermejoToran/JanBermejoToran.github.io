+++
date = '2026-11-09T09:33:57+01:00'
draft = false
title = 'CyberHeroes THM Writeup'
summary = 'Writeup of the CyberHeroes machine on TryHackMe'
tags = ["THM", "web", "writeup", "authentication-bypass", "client-side", "Red-Team"]
+++


![CyberHeroes machine logo](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/CyberHeroes-THM-Logo.png)

# Introduction

These are the details of this TryHackMe machine called CyberHeroes:

- Platform: TryHackMe
- Machine: CyberHeroes
- Difficulty: Easy
- Category: Web
- Estimated time: 15 minutes

Link: [CyberHeroes - TryHackMe](https://tryhackme.com/room/cyberheroes)

## Challenge description

Want to be part of the CyberHeroes elite club? Prove your worth by finding a way to log in!


Access this challenge by deploying both the vulnerable machine by pressing the green "Start Lab Machine" button in this task, and the TryHackMe AttackBox by pressing the "Start AttackBox" button at the top right of the page.

Go to the following URL using the AttackBox: http://MACHINE_IP


Check out similar content on TryHackMe:

- Authentication bypass

## Reconnaissance

The first thing we are going to do is run an nmap scan against the IP of the vulnerable machine:

```bash
[x@x ~]$ nmap -F 10.128.191.43
Starting Nmap 7.993 ( https://nmap.org ) at 2026-11-09 09:40 +0100
Nmap scan report for 10.128.191.43
Host is up (0.053s latency).
Not shown: 98 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 2.04 seconds
```
![Screenshot of the nmap port scan against the IP](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Captura_nmap_CyberHeroes-THM.png)

We can see it has HTTP, which means it probably hosts a website, so let's put the IP in our browser to check it out:

![Main page of the website being hosted](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Pagina_inicio-CyberHeroes-THM.png)

The site has three sections:

- Home
- About
- Login

This is the About page:

![About page of the CyberHeroes website](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/About-CyberHeroes-THM.png)

And this is the login:

![Login page of this website](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Login-CyberHeroes-THM.png)

Okay, now that we have found the login, let's figure out how to exploit it so we can log in as the challenge asks.

## Vulnerability identification

The first thing we are going to do is look at the source code of login.html to see if we can find anything about how it works. To do that, press ctrl+u

![CyberHeroes website source code](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/codigo_Web-CyberHeroes-THM.png)

After reading through all this code, I found a part that particularly caught my attention. It is the following:

![Interesting part of the code](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

Let's work out what this block says

![Specific CyberHeroes block](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

As we can see, this seems to be the credential validation system, and a pretty insecure one, since it shows us the required username as well as a reversestring, which is the password written backwards

![Important parts of the website code highlighted](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_secciones_importante_codigo-CyberHeroes-THM.png)

With this block, which seems to be in charge of this login's authentication, and knowing that the script shows what looks like a username and a password, we are going to move on to exploitation.

### Vulnerability found

The vulnerability is **client-side authentication** with **hardcoded credentials** in the source code. It maps to these categories:

- CWE-603: Use of Client-Side Authentication
- CWE-798: Use of Hard-coded Credentials

### Why it is vulnerable

It is vulnerable because the validation is performed in the browser, using a script with the credentials hardcoded in it. In addition, the server takes no part in the check.


## Exploitation

First of all, we can see that the supposed password is stored as a ReverseString, which means the string is reversed. So we are going to flip the text back, and there are several ways to do it:

Website:

We can go to this website and paste the password directly into the blue box, which will give us the result straight away

https://onlinestringtools.com/reverse-string

![ReverseString solved](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/ReverseString_Resuelta-CyberHeroes-THM.png)

Terminal:

It can also be done directly from the terminal with the `rev` command, without relying on third-party websites (it is better not to paste credentials into external sites):

```bash
echo 'reversed_string' | rev
```

Replacing `reversed_string` with the string that appears in the code, the command returns the original password.


![String reversed in the terminal](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/reverse_string_terminal-CyberHeroes-THM.png)

And with these credentials we are going to try to log in to the website.

![CyberHeroes login attempt](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/prueba_login-CyberHeroes-THM.png)

We enter the credentials, they work, and I get in.

![Successful login + flag](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)

And with this we already have the flag.

## Getting the flag

And with this we have the flag for this machine, which we obtained thanks to an authentication script exposed in the source code of this website.

![Flag for this level](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)


## Thanks for reading.