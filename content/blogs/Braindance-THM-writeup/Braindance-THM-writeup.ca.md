+++
date = '2026-11-20T16:23:10+01:00'
draft = false
title = 'Braindance THM Writeup'
summary = 'Writeup de Braindance de TryHackMe'
tags = ["THM", "web", "writeup"]
+++

![Logo de la màquina Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Braindance_logo.png)

# Introducció

Aquestes són les característiques d'aquesta màquina de TryHackMe anomenada Braindance:

- Plataforma: TryHackMe
- Màquina: Braindance
- Dificultat: Easy
- Categoria: Web
- Durada estimada: 20m

Link: [Quan la màquina estigui publicada]()

## Enunciat

Braindance és una petita botiga de "neurodances": clips de ball que es carreguen directament al teu còrtex motor. El negoci va bé, però darrere el taulell han anat deixant caps solts. Aconsegueix la user flag i la root flag.

## Reconeixement

El primer, un escaneig de ports a la ip de la màquina.

```tmux
nmap -sC -sV MACHINE_IP
```

![Escaneig de ports de la màquina Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/Escaneo_puertos_Braindance-THM.png)

Només hi ha dos ports oberts: el 22 (SSH) i el 80 (Apache). Amb el 80 obert, el primer és entrar pel navegador i veure què trobem.

![Captura de la web de la botiga Braindance](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/web-Braindance-THM.png)

És un quiosc de lloguer de clips, amb un catàleg a la dreta. Li faig un cop d'ull per sobre a la pàgina, sense tocar res encara, només per veure quin tipus de contingut maneja.

## Identificació de la vulnerabilitat

Després de mirar la web per sobre, obro les eines de desenvolupador i vaig a la pestanya Network, perquè vull veure què passa per darrere quan interactuo amb la pàgina.

![Pestanya Network oberta abans d'interactuar amb el catàleg](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/inspector_red-Braindance-THM.png)

Premo sobre un dels clips del catàleg i veig que es dispara una petició:

![Petició que es dispara en prémer un clip del catàleg](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/interesante_inspector_red-Braindance-THM.png)

Ara obro bé la petició i veig que es fa la sol·licitud a `/api/dance.php?id=2`. Cada clip del catàleg té el seu propi id (del 2 al 6, que són els que es veuen llistats), així que el lògic és provar si existeix algun id que no estigui enllaçat a la web. Provo directament amb l'`id=1`.

![Informació de la petició a l'apartat de xarxa](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/peticion_api-Braindance-THM.png)

```tmux
curl http://MACHINE_IP/api/dance.php?id=1
```

![Resposta de l'API en demanar l'id 1, amb credencials exposades](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/curl_id1-Braindance-THM.png)

I aquí ho tenim: l'id 1 és un clip que mai s'hauria d'haver publicat, i al camp de notes internes porta l'usuari i la contrasenya del compte de manteniment. L'API no comprova que el clip demanat estigui realment publicat, així que qualsevol pot demanar un id que no apareix al catàleg. Això és un IDOR (Insecure Direct Object Reference): el control d'accés depèn que l'id no es mostri a la interfície, no que el servidor el validi.

## Explotació

Amb les credencials que he tret de l'API, provo de connectar-me per SSH.

```tmux
ssh braindancer@MACHINE_IP
```

![Connexió SSH reeixida amb les credencials filtrades](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/ssh-Braindance-THM.png)

Entro sense problema. Reviso la carpeta de l'usuari i aquí tenim la primera flag.

```tmux
ls
cat user.txt
```

![Contingut de user.txt censurat](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/flag_user-Braindance-THM.png)

## Escalada de privilegis

Ja dins, el primer que miro són els permisos de sudo de l'usuari.

```tmux
sudo -l
```

![Sortida de sudo -l mostrant el binari vim permès](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/sudo_l-Braindance-THM.png)

L'usuari pot executar `/usr/bin/vim` com a root sense contrasenya. Vim és un dels binaris clàssics de GTFOBins per escalar privilegis: un cop dins, es pot llançar una shell directament des del mateix editor.

```tmux
sudo vim -c ':!/bin/bash'
```

Resumidament el que fa aquesta comanda és el següent:

- `sudo vim`: obre vim amb permisos de root, que és just el que ens permet la regla de sudo.
- `-c ':!/bin/bash'`: li diu a vim que executi una comanda d'Ex just en obrir-se, en aquest cas llançar una shell de bash.
- Com que vim s'ha obert com a root, la shell que llança també hereta aquests permisos.

![Shell de root obtinguda des de vim](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/escalada_vim-Braindance-THM.png)

## Obtenció de la flag

I amb la shell de root ja puc llegir la segona flag.

```tmux
cd /root/
ls
cat /root/root.txt
```

![La root flag d'aquesta màquina censurada](/images/writeups/TryHackMe_imágenes/Braindance-THM-imágenes/captura_flag_root-Braindance-THM.png)

## Gràcies per llegir.