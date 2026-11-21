+++
date = '2026-11-20T16:23:10+01:00'
draft = false
title = 'Braindance THM Writeup'
summary = 'Writeup of Braindance from TryHackMe'
tags = ["THM", "web", "writeup"]
+++

![Logo of the Braindance machine](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Braindance_logo.png)

# Introduction

These are the characteristics of this TryHackMe machine called Braindance:

- Platform: TryHackMe
- Machine: Braindance
- Difficulty: Easy
- Category: Web
- Estimated time: 20m

Link: [Once the machine is published]()

## Prompt

Braindance is a small "neurodance" shop: dance clips loaded straight into your motor cortex. Business is good, but behind the counter they've left a few loose ends. Get the user flag and the root flag.

## Reconnaissance

First, a port scan against the machine's ip.

```tmux
nmap -sC -sV MACHINE_IP
```

![Port scan of the Braindance machine](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Escaneo_puertos_Braindance-THM.png)

Only two ports are open: 22 (SSH) and 80 (Apache). With port 80 open, the first move is to open it in the browser and see what we find.

![Screenshot of the Braindance shop website](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/web-Braindance-THM.png)

It's a clip rental kiosk, with a catalog on the right. I give the page a quick look over, without touching anything yet, just to see what kind of content it handles.

## Identifying the vulnerability

After looking the site over, I open the developer tools and go to the Network tab, because I want to see what's happening behind the scenes when I interact with the page.

![Network tab open before interacting with the catalog](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/inspector_red-Braindance-THM.png)

I click on one of the clips in the catalog and see a request fire off:

![Request fired when clicking a clip in the catalog](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/interesante_inspector_red-Braindance-THM.png)

I open the request properly and see it's calling `/api/dance.php?id=2`. Each clip in the catalog has its own id (2 through 6, the ones shown in the listing), so the obvious move is to try an id that isn't linked anywhere on the site. I try `id=1` directly.

![Request details in the network panel](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/peticion_api-Braindance-THM.png)

```tmux
curl http://MACHINE_IP/api/dance.php?id=1
```

![API response when requesting id 1, with exposed credentials](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/curl_id1-Braindance-THM.png)

And there it is: id 1 is a clip that was never meant to go live, and its internal notes field carries the username and password for the maintenance account. The API never checks whether the requested clip is actually published, so anyone can request an id that doesn't show up in the catalog. This is an IDOR (Insecure Direct Object Reference): access control relies on the id not being shown in the interface, not on the server actually validating it.

## Exploitation

With the credentials pulled from the API, I try connecting over SSH.

```tmux
ssh braindancer@MACHINE_IP
```

![Successful SSH login with the leaked credentials](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/ssh-Braindance-THM.png)

I get in without issue. I check the user's home folder and there's the first flag.

```tmux
ls
cat user.txt
```

![Contents of user.txt, redacted](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/flag_user-Braindance-THM.png)

## Privilege escalation

Once inside, the first thing I check is the user's sudo permissions.

```tmux
sudo -l
```

![Output of sudo -l showing vim is allowed](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/sudo_l-Braindance-THM.png)

The user can run `/usr/bin/vim` as root with no password. Vim is one of the classic GTFOBins binaries for privilege escalation: once inside it, you can spawn a shell directly from the editor itself.

```tmux
sudo vim -c ':!/bin/bash'
```

Here's a quick breakdown of what this command does:

- `sudo vim`: opens vim with root privileges, which is exactly what the sudo rule allows.
- `-c ':!/bin/bash'`: tells vim to run an Ex command right after opening, in this case spawning a bash shell.
- Since vim was opened as root, the shell it spawns inherits those same privileges.

![Root shell obtained from vim](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/escalada_vim-Braindance-THM.png)

## Getting the flag

And with the root shell, I can now read the second flag.

```tmux
cd /root/
ls
cat /root/root.txt
```

![The root flag for this machine, redacted](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/captura_flag_root-Braindance-THM.png)

## Thanks for reading.