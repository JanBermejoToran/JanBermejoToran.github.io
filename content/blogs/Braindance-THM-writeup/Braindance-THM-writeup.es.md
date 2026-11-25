+++
date = '2026-11-20T16:23:10+01:00'
draft = false
title = 'Braindance THM Writeup'
summary = 'Writeup de Braindance de TryHackMe'
tags = ["THM", "web", "writeup"]
+++


![Logo de la máquina Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Braindance_logo.png)

# Introducción

Estas son las características de esta máquina de TryHackMe llamada Braindance:

- Plataforma: TryHackMe
- Máquina: Braindance
- Dificultad: Easy
- Categoría: Web
- Duración estimada: 20m

Link: [Cuando la maquina este publica]()

## Enunciado

Braindance es una pequeña tienda de "neurodanzas": clips de baile que se cargan directamente a tu corteza motora. El negocio va bien, pero por detrás del mostrador han ido dejando cabos sueltos. Consigue la user flag y la root flag.

## Reconocimiento

Lo primero, un escaneo de puertos a la ip de la máquina.

```bash
nmap -sC -sV MACHINE_IP
```

![Escaneo de puertos de la máquina Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Escaneo_puertos_Braindance-THM.png)

Solo hay dos puertos abiertos: el 22 (SSH) y el 80 (Apache). Con el 80 abierto, lo primero es entrar por el navegador y ver qué nos encontramos.

![Captura de la web de la tienda Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/web-Braindance-THM.png)

Es un kiosco de alquiler de clips, con un catálogo a la derecha. Le doy un repaso por encima a la página, sin tocar nada todavía, solo para ver qué tipo de contenido maneja.

## Identificación de la vulnerabilidad

Después de mirar la web por encima, abro las herramientas de desarrollador y me voy a la pestaña Network, porque quiero ver qué está pasando por detrás cuando interactúo con la página.

![Pestaña Network abierta antes de interactuar con el catálogo](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/inspector_red-Braindance-THM.png)

Pulso sobre uno de los clips del catálogo y veo que se dispara una petición:

![Petición que se dispara al pulsar un clip del catálogo](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/interesante_inspector_red-Braindance-THM.png)

Ahora abro bien la petición y veo que se hace la solicitud a `/api/dance.php?id=2`.

![Información de la petición en el apartado de red](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/peticion_api-Braindance-THM.png)

Cada clip del catálogo tiene su propio id (del 2 al 6, que son los que se ven listados), así que lo lógico es probar si existe algún id que no esté enlazado en la web. Pruebo directamente con el `id=1`.

```bash
curl http://MACHINE_IP/api/dance.php?id=1
```

![Respuesta de la API al pedir el id 1, con credenciales expuestas](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/curl_id1-Braindance-THM.png)

Y ahí está: el id 1 es un clip que nunca debería haberse publicado, y en el campo de notas internas lleva el usuario y la contraseña de la cuenta de mantenimiento. La API no comprueba que el clip pedido esté realmente publicado, así que cualquiera puede pedir un id que no aparece en el catálogo. Esto es un IDOR (Insecure Direct Object Reference): el control de acceso depende de que el id no se muestre en la interfaz, no de que el servidor lo valide.

## Explotación

Con las credenciales que he sacado de la API, pruebo a conectarme por SSH.

```bash
ssh braindancer@MACHINE_IP
```

![Conexión SSH exitosa con las credenciales filtradas](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/ssh-Braindance-THM.png)

Entro sin problema. Reviso la carpeta del usuario y ahí está la primera flag.

```bash
ls
cat user.txt
```

![Contenido de user.txt censurado](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/flag_user-Braindance-THM.png)

## Escalada de privilegios

Ya dentro, lo primero que miro son los permisos de sudo del usuario.

```bash
sudo -l
```

![Salida de sudo -l mostrando el binario vim permitido](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/sudo_l-Braindance-THM.png)

El usuario puede ejecutar `/usr/bin/vim` como root sin contraseña. Vim es uno de los binarios clásicos de GTFOBins para escalar privilegios: una vez dentro, se puede lanzar una shell directamente desde el propio editor.

```bash
sudo vim -c ':!/bin/bash'
```

Resumidamente lo que hace este comando es lo siguiente:

- `sudo vim`: abre vim con permisos de root, que es justo lo que nos permite la regla de sudo.
- `-c ':!/bin/bash'`: le dice a vim que ejecute un comando de Ex nada más abrir, en este caso lanzar una shell de bash.
- Como vim se ha abierto como root, la shell que lanza también hereda esos permisos.

![Shell de root obtenida desde vim](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/escalada_vim-Braindance-THM.png)

## Obtención de la flag

Y con la shell de root ya puedo leer la segunda flag.

```bash
cd /root/
ls
cat /root/root.txt
```

![La root flag de esta máquina censurada](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/captura_flag_root-Braindance-THM.png)

## Gracias por leer.