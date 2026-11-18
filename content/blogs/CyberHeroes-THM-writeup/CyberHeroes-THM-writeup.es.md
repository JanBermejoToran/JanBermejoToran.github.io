+++
date = '2026-11-09T09:33:57+01:00'
draft = false
title = 'CyberHeroes THM Writeup'
summary = 'Writeup de la máquina CyberHeroes de TryHackMe'
tags = ["THM", "web", "writeup", "authentication-bypass", "client-side", "Red-Team"]
+++


![Logo de la máquina CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/CyberHeroes-THM-Logo.png)

# Introducción

Estas son las características de esta máquina de TryHackMe llamada CyberHeroes:

- Plataforma: TryHackMe
- Máquina: CyberHeroes
- Dificultad: Easy
- Categoría: Web
- Duración estimada: 15 minutos

Link: [CyberHeroes - TryHackMe](https://tryhackme.com/room/cyberheroes)

## Enunciado

¿Quieres formar parte del club de élite de CyberHeroes? ¡Demuestra tu mérito encontrando una manera de iniciar sesión!


Acceda a este desafío implementando tanto la máquina vulnerable presionando el botón verde "Start Lab Machine" ubicado dentro de esta tarea, como el TryHackMe AttackBox presionando el botón "Iniciar AttackBox" ubicado en la parte superior derecha de la página.

Vaya a la siguiente URL usando el AttackBox: http///MACHINE_IP


Echa un vistazo a contenido similar en TryHackMe:

- Bypass de autenticación

## Reconocimiento

Lo primero que vamos a hacer va a ser un nmap a la IP de la máquina vulnerable:

```tmux
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
![Captura del escaneo de puertos con nmap a la ip](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Captura_nmap_CyberHeroes-THM.png)

Vemos que tiene http, eso quiere decir que probablemente tenga una web, así que vamos a poner la IP en nuestro navegador para buscarla:

![Página principal de la página que está hosteando](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Pagina_inicio-CyberHeroes-THM.png)

Vemos que tiene una página, la cual tiene tres apartados:

- Home
- About
- Login

Este es el about:

![About de la página web de CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/About-CyberHeroes-THM.png)

Y este es el login:

![Login de esta página web](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Login-CyberHeroes-THM.png)

Okey, y ahora que hemos encontrado el login, lo que vamos a hacer es averiguar cómo vulnerarlo para poder loguearnos como nos pide el enunciado.

## Identificación de la vulnerabilidad

Lo primero que vamos a hacer es ver el código de login.html para ver si podemos encontrar algo de cómo funciona, para ello vamos a darle a ctrl+u

![Código de la web de CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/codigo_Web-CyberHeroes-THM.png)

Y una vez leído todo este código, he encontrado una parte que me llama especialmente la atención, es la siguiente:

![Parte interesante del código](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

Vamos a descifrar qué pone en este bloque

![Bloque concreto CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_concreto-CyberHeroes-THM.png)

Si vemos, parece ser que es el sistema de validación de credenciales, uno bastante inseguro, ya que nos muestra el usuario que necesita además de una reversestring la cual es la contraseña invertida

![Partes del código importantes resaltadas en el código de la web](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Bloque_secciones_importante_codigo-CyberHeroes-THM.png)

Con este bloque que parece ser el encargado de la autenticación de este login, y sabiendo que en el script vemos lo que parece un usuario y una contraseña, lo que vamos a hacer será pasar a la explotación.

### Vulnerabilidad encontrada

La vulnerabilidad es una **autenticación del lado del cliente (client-side authentication)** con **credenciales hardcodeadas** en el código fuente. Se corresponde con estas categorías:

- CWE-603: Use of Client-Side Authentication
- CWE-798: Use of Hard-coded Credentials

### Por qué es vulnerable

Es vulnerable por que la validación la hace desde el navegador, usando un script con las credenciales hardcodeadas, ademas, el servidor no participa en la comprobación.


## Explotación

Lo primero de todo, vemos que la supuesta contraseña está en una ReverseString, eso quiere decir que la String está revertida, con lo cual, vamos a darle la vuelta al texto, para ello podemos hacerlo de varias formas:

Página web:

Podemos entrar en esta pagina web y pegar la contraseña directamente en la caja azul, donde nos dara el resultado directamente

https://onlinestringtools.com/reverse-string

![ReverseString resuelta](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/ReverseString_Resuelta-CyberHeroes-THM.png)

Terminal:

También se puede hacer directamente desde la terminal con el comando `rev`, sin depender de páginas de terceros (conviene no pegar credenciales en webs externas):

```bash
echo 'cadena_invertida' | rev
```

Sustituyendo `cadena_invertida` por la cadena que aparece en el código, el comando devuelve la contraseña original.


![Cadena revertida en terminal](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/reverse_string_terminal-CyberHeroes-THM.png)

Y con estas credenciales vamos a probar a loguearnos en la página.

![Prueba login CyberHeroes](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/prueba_login-CyberHeroes-THM.png)

Y introducimos las credenciales, las cuales han funcionado y me dejan entrar.

![Login correcto + flag](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)

Y con esto ya tenemos la flag.

## Obtención de la flag

Y con esto ya tenemos la flag de esta máquina, la cual la hemos obtenido gracias a un script de autenticación expuesto en el código fuente de esta página web.

![Flag de este nivel](/images/writeups/TryHackMe_imágenes/CyberHeroes-THM-imágenes/Flag_Resuelta-CyberHeroes-THM.png)


## Gracias por leer.