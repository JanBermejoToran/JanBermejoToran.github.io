+++
date = '2026-11-06T09:31:27+01:00'
draft = false
title = 'Neighbour THM Writeup'
summary = 'Writeup de la máquina Neighbour de TryHackMe'
tags = ["THM", "web", "writeup", "IDOR"]
+++

![Logo de la máquina Neighbour](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Neighbour_logo.png)

# Introducción 

Estas son las características de esta máquina de TryHackMe llamada Neighbour:

- Plataforma: TryHackMe 
- Máquina: Neighbour
- Dificultad: Easy
- Categoría: Web

Link: [Neighbour - TryHackMe](https://tryhackme.com/room/neighbour)

## Enunciado

Echa un vistazo a nuestro nuevo servicio en la nube, Authentication Anywhere - ¡iniciar sesión desde cualquier lugar que desee! Los usuarios pueden introducir su nombre de usuario y contraseña, ¡para un proceso de inicio de sesión totalmente seguro! Definitivamente no podrías encontrar ningún secreto que otras personas tengan en su perfil, ¿verdad?


Acceda a este desafío implementando tanto la máquina vulnerable presionando el botón verde "Start Lab Machine" ubicado dentro de esta tarea, como el TryHackMe AttackBox presionando el botón "Iniciar AttackBox" ubicado en la parte superior derecha de la página.

Vaya a la siguiente URL usando el AttackBox: http://MACHINE_IP


Echa un vistazo a contenido similar en TryHackMe:

- IDOR

## Reconocimiento

Para esta máquina lo que vamos a hacer va a ser acceder a la web de la que nos están hablando:

![Captura de cómo se ve la página web de esta máquina](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_web-Neighbour-THM.png)


Okey, vemos cómo nos piden que iniciemos sesión en esta página web, pero no sabemos ni el usuario ni la contraseña, pero vemos que nos dice que si no tenemos usuario ni contraseña podemos usar la cuenta de guest:

![Pista que nos da la misma página](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_Pista-Neighbour-THM.png)

La página nos recomienda usar la cuenta de invitado, pero no dice la contraseña. Vamos a mirar el código fuente con Ctrl+U, por si hay alguna pista.

![Código de la página web](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_codigo_web_Neighbour-THM.png)

Ahora que nos hemos leído el código, podemos ver que tiene esta sección de aquí que pone que podemos iniciar sesión usando como usuario y contraseña unas credenciales que han quedado expuestas en el código.

![Captura de la sección donde sale que podemos iniciar como guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_codigo_guest-Neighbour-THM.png)

Con el usuario y contraseña, lo que vamos a hacer va a ser ir de vuelta al login y vamos a poner estos datos.

![Login con guest como usuario y contraseña guest:guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Login_guest_Neighbour-THM.png)

Perfecto, como podemos ver, hemos iniciado sesión como guest, y con esto podemos ver varias cosas, para empezar el mensaje, que en español significa: "Hola, invitado. Bienvenido a nuestro sitio. Trata de no mirar el perfil de tu vecino."

![Login exitoso en guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_dentro_guest_Neighbour-THM.png)

Y estando ya dentro, nos podemos fijar en la forma que utilizan para estar dentro del usuario guest:

![URL de la página estando ya iniciados sesión como guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_Neighbour-THM.png)

Con esto podemos pasar ya a la identificación de vulnerabilidades


## Identificación de la vulnerabilidad

La principal cosa que me ha llamado la atención de esta máquina ha sido la URL del usuario guest cuando ya hemos iniciado sesión, el final, donde pone guest.

![Identificador de usuario](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Identificador_usuario_Neighbour-THM.png)

Con esto sospecho que se trata de una vulnerabilidad IDOR, con la cual podríamos saltar al usuario admin simplemente cambiando la URL de tal forma que en vez de ver la página de user, se vea la página de admin.


## Explotación

Para la explotación de lo que sospecho que es esta IDOR simplemente debemos cambiar la URL para que en vez de tener user=guest sea user=admin.

---

#### ¿Qué es un IDOR?

IDOR (Insecure Direct Object Reference) es una vulnerabilidad de control de acceso que ocurre cuando una aplicación usa un identificador controlado por el usuario (como `user=guest` en la URL) para acceder a una página u otro recurso, sin comprobar si ese usuario tiene permiso para verlo. Al modificar el identificador, podemos acceder a datos de otros usuarios.

---

Así que cambiamos eso en nuestra URL:

![Cambiando la URL para que en vez de guest ponga admin para el IDOR](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_IDOR-Neighbour-THM.png)

Y con esto le damos a enter para ir a la página:

![Captura de la página de admin](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

¡Perfecto! Con esto hemos podido entrar en la página del usuario administrador y hemos conseguido la flag.

## Obtención de la flag

Y con esto ya tenemos la flag de esta máquina, la cual la hemos obtenido gracias a un IDOR en esta página web.

![Flag de este nivel](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)


## Gracias por leer.