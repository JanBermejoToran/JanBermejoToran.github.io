+++
date = '2026-11-06T09:31:27+01:00'
draft = false
title = 'Neighbour THM Writeup'
summary = 'Writeup of the TryHackMe machine Neighbour'
tags = ["THM", "web", "writeup", "IDOR"]
+++

![Neighbour machine logo](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Neighbour_logo.png)

# Introduction 

These are the characteristics of this TryHackMe machine called Neighbour:

- Platform: TryHackMe 
- Machine: Neighbour
- Difficulty: Easy
- Category: Web

## Challenge description

Check out our new cloud service, Authentication Anywhere - log in from wherever you want! Users can enter their username and password, for a totally secure login process! You definitely couldn't find any secrets other people have in their profile, right?


Access this challenge by deploying both the vulnerable machine by pressing the green "Start Lab Machine" button located inside this task, and the TryHackMe AttackBox by pressing the "Start AttackBox" button located at the top right of the page.

Go to the following URL using the AttackBox: http://MACHINE_IP


Check out similar content on TryHackMe:

- IDOR

## Reconnaissance

For this machine, what we are going to do is access the website they are talking about:

![Screenshot of what this machine's website looks like](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_web-Neighbour-THM.png)


Okay, we can see they ask us to log in on this website, but we know neither the username nor the password, but we see it tells us that if we don't have a username or password we can use the guest account:

![Hint given by the page itself](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_Pista-Neighbour-THM.png)

So let's do it, since basically using ctrl+u lets us see the page's source code:

![Website source code](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_codigo_web_Neighbour-THM.png)

Now that we have read the code, we can see it has this section here saying we can log in using guest as both username and password:

![Screenshot of the section showing we can log in as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_codigo_guest-Neighbour-THM.png)

With the username and password, what we are going to do is go back to the login and enter these credentials.

![Login with guest as username and password guest:guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Login_guest_Neighbour-THM.png)

Perfect, as we can see, we have logged in as guest, and with this we can see several things, starting with the message, which says: "Hi, guest. Welcome to our site. Try not to peep at your neighbor's profile."

![Successful login as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_dentro_guest_Neighbour-THM.png)

And now that we are inside, we can notice how they identify us inside the guest user:

![Page URL once logged in as guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_Neighbour-THM.png)

With this we can move on to identifying vulnerabilities


## Vulnerability identification

The main thing that caught my attention about this machine was the guest user's URL once we have logged in, the end, where it says guest.

![User identifier](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Identificador_usuario_Neighbour-THM.png)

With this I suspect it is an IDOR vulnerability, with which we could jump to the admin user simply by changing the URL so that instead of seeing the user page, we see the admin page.


## Exploitation

To exploit what I suspect is an IDOR, we simply need to change the URL so that instead of user=guest it says user=admin.

---

#### What is an IDOR?

IDOR (Insecure Direct Object Reference) is an access control vulnerability that occurs when an application uses a user-controlled identifier (such as `user=guest` in the URL) to access a page or other resource, without checking whether that user has permission to view it. By modifying the identifier, we can access other users' data.

---

So we change that in our URL:

![Changing the URL so it says admin instead of guest for the IDOR](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_IDOR-Neighbour-THM.png)

And with this we press enter to go to the page:

![Screenshot of the admin page](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

Perfect! With this we have managed to enter the administrator user's page and we have got the flag.

## Getting the flag

And with this we now have this machine's flag, which we obtained thanks to an IDOR on this website.

![Flag of this level](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)


## Thanks for reading.