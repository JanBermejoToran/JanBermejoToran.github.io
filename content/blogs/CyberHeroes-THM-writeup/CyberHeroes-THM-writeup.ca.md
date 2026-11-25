+++
date = '2026-11-09T09:33:57+01:00'
draft = false
title = 'CyberHeroes THM Writeup'
summary = 'Writeup de la màquina CyberHeroes de TryHackMe'
tags = ["THM", "web", "writeup", "authentication-bypass", "client-side", "Red-Team"]
+++


![Logotip de la màquina CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/CyberHeroes-THM-Logo.png)

# Introducció

Aquestes són les característiques d'aquesta màquina de TryHackMe anomenada CyberHeroes:

- Plataforma: TryHackMe
- Màquina: CyberHeroes
- Dificultat: Easy
- Categoria: Web
- Durada estimada: 15 minuts

Link: [CyberHeroes - TryHackMe](https://tryhackme.com/room/cyberheroes)

## Enunciat

Vols formar part del club d'elit de CyberHeroes? Demostra el teu mèrit trobant una manera d'iniciar sessió!


Accedeix a aquest repte desplegant tant la màquina vulnerable prement el botó verd "Start Lab Machine" situat dins d'aquesta tasca, com el TryHackMe AttackBox prement el botó "Iniciar AttackBox" situat a la part superior dreta de la pàgina.

Ves a la URL següent fent servir l'AttackBox: http://MACHINE_IP


Fes un cop d'ull a contingut similar a TryHackMe:

- Bypass d'autenticació

## Reconeixement

El primer que farem serà un nmap a la IP de la màquina vulnerable:

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
![Captura de l'escaneig de ports amb nmap a la IP](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Captura_nmap_CyberHeroes-THM.png)

Veiem que té http, això vol dir que probablement tingui una web, així que posarem la IP al navegador per buscar-la:

![Pàgina principal de la pàgina que està allotjant](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Pagina_inicio-CyberHeroes-THM.png)

Veiem que té una pàgina, la qual té tres apartats:

- Home
- About
- Login

Aquest és l'about:

![About de la pàgina web de CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/About-CyberHeroes-THM.png)

I aquest és el login:

![Login d'aquesta pàgina web](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Login-CyberHeroes-THM.png)

D'acord, i ara que hem trobat el login, el que farem serà esbrinar com vulnerar-lo per poder iniciar sessió com ens demana l'enunciat.

## Identificació de la vulnerabilitat

El primer que farem és mirar el codi de login.html per veure si podem trobar alguna cosa de com funciona. Per fer-ho, premem ctrl+u

![Codi de la web de CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/codigo_Web-CyberHeroes-THM.png)

I un cop llegit tot aquest codi, he trobat una part que em crida especialment l'atenció, és la següent:

![Part interessant del codi](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

Desxifrem què diu aquest bloc

![Bloc concret de CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

Si ho mirem, sembla que és el sistema de validació de credencials, un de bastant insegur, ja que ens mostra l'usuari que cal a més d'una reversestring, la qual és la contrasenya invertida

![Parts importants del codi ressaltades al codi de la web](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_secciones_importante_codigo-CyberHeroes-THM.png)

Amb aquest bloc, que sembla que és l'encarregat de l'autenticació d'aquest login, i sabent que a l'script veiem el que sembla un usuari i una contrasenya, passarem a l'explotació.

### Vulnerabilitat trobada

La vulnerabilitat és una **autenticació del costat del client (client-side authentication)** amb **credencials hardcodejades** al codi font. Es correspon amb aquestes categories:

- CWE-603: Use of Client-Side Authentication
- CWE-798: Use of Hard-coded Credentials

### Per què és vulnerable

És vulnerable perquè la validació es fa des del navegador, fent servir un script amb les credencials hardcodejades. A més, el servidor no participa en la comprovació.


## Explotació

Primer de tot, veiem que la suposada contrasenya està en una ReverseString, això vol dir que la String està revertida, així que li donarem la volta al text, i per fer-ho ho podem fer de diverses maneres:

Pàgina web:

Podem entrar en aquesta pàgina web i enganxar la contrasenya directament a la caixa blava, on ens donarà el resultat directament

https://onlinestringtools.com/reverse-string

![ReverseString resolta](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/ReverseString_Resuelta-CyberHeroes-THM.png)

Terminal:

També es pot fer directament des del terminal amb l'ordre `rev`, sense dependre de pàgines de tercers (convé no enganxar credencials a webs externes):

```bash
echo 'cadena_invertida' | rev
```

Substituint `cadena_invertida` per la cadena que apareix al codi, l'ordre retorna la contrasenya original.


![Cadena revertida al terminal](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/reverse_string_terminal-CyberHeroes-THM.png)

I amb aquestes credencials provarem d'iniciar sessió a la pàgina.

![Prova de login a CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/prueba_login-CyberHeroes-THM.png)

I introduïm les credencials, les quals han funcionat i em deixen entrar.

![Login correcte + flag](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)

I amb això ja tenim la flag.

## Obtenció de la flag

I amb això ja tenim la flag d'aquesta màquina, la qual hem obtingut gràcies a un script d'autenticació exposat al codi font d'aquesta pàgina web.

![Flag d'aquest nivell](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)


## Gràcies per llegir.