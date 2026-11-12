+++
date = '2026-11-06T09:31:27+01:00'
draft = false
title = 'Neighbour THM Writeup'
summary = 'Writeup of the Neighbour room from TryHackMe'
tags = ["THM", "web", "writeup", "IDOR"]
+++

![Neighbour machine logo](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Neighbour_logo.png)

# Introduction

These are the details of this TryHackMe machine called Neighbour:

- Platform: TryHackMe
- Machine: Neighbour
- Difficulty: Easy
- Category: Web

Link: [Neighbour - TryHackMe](https://tryhackme.com/room/neighbour)

## Challenge description

Check out our new cloud service, Authentication Anywhere - log in from anywhere you want! Users can enter their username and password for a completely secure login process! You definitely couldn't find any secrets other people have in their profile, right?

Access this challenge by deploying both the vulnerable machine by pressing the green "Start Lab Machine" button located within this task, and the TryHackMe AttackBox by pressing the "Start AttackBox" button located at the top-right of the page.

Navigate to the following URL using the AttackBox: http://MACHINE_IP

Check out similar content on TryHackMe:

- IDOR

## Reconnaissance

For this machine, the first thing we are going to do is access the website they are talking about:

![Screenshot of the machine's website](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_web-Neighbour-THM.png)

Okay, we can see that we are asked to log in to this website, but we know neither the username nor the password. However, the page tells us that if we don't have an account we can use the guest account:

![Hint given by the page itself](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_Pista-Neighbour-THM.png)

The page recommends using the guest account, but it doesn't tell us the password. Let's look at the source code with Ctrl+U, in case there is a clue.

![Source code of the web page](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_codigo_web_Neighbour-THM.png)

Now that we have read the code, we can see this section, which tells us we can log in using credentials that have been exposed in the code.

![Screenshot of the section showing we can log in as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_codigo_guest-Neighbour-THM.png)

With the username and password, we are going to go back to the login page and enter them.

![Login with the guest account](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Login_guest_Neighbour-THM.png)

Perfect, as we can see, we have logged in as guest, and there are several things we can notice. To begin with, the message: "Hi, guest. Welcome to our site. Try not to peep your neighbor's profile."

![Successful login as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_dentro_guest_Neighbour-THM.png)

And now that we are inside, we can look at how the site identifies the guest user:

![Page URL once logged in as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_Neighbour-THM.png)

With this, we can move on to identifying vulnerabilities.

## Vulnerability identification

The main thing that caught my attention in this machine was the guest user's URL once logged in, specifically the end, where it says guest.

![User identifier](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Identificador_usuario_Neighbour-THM.png)

From this I suspect it is an IDOR vulnerability, with which we could jump to the admin user simply by changing the URL so that, instead of showing the user page, it shows the admin page.

## Exploitation

To exploit what I suspect is an IDOR, we simply need to change the URL so that instead of user=guest it says user=admin.

---

#### What is an IDOR?

IDOR (Insecure Direct Object Reference) is an access control vulnerability that occurs when an application uses a user-controlled identifier (such as `user=guest` in the URL) to access a page or other resource without checking whether that user has permission to view it. By modifying the identifier, we can access other users' data.

---

So we change that in our URL:

![Changing the URL so it says admin instead of guest for the IDOR](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_IDOR-Neighbour-THM.png)

And we press Enter to go to the page:

![Screenshot of the admin page](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

Perfect! We have managed to enter the administrator user's page and we have obtained the flag.

## Obtaining the flag

And with this we have the flag for this machine, which we obtained thanks to an IDOR in this website.

![Flag of this level](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

## Thanks for reading.