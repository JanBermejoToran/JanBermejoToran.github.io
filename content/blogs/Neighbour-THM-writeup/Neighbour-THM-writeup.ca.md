+++
date = '2026-11-06T09:31:27+01:00'
draft = false
title = 'Neighbour THM Writeup'
summary = 'Writeup de la màquina Neighbour de TryHackMe'
tags = ["THM", "web", "writeup", "IDOR"]
+++

![Logo de la màquina Neighbour](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Neighbour_logo.png)

# Introducció

Aquestes són les característiques d'aquesta màquina de TryHackMe anomenada Neighbour:

- Plataforma: TryHackMe
- Màquina: Neighbour
- Dificultat: Easy
- Categoria: Web

Enllaç: [Neighbour - TryHackMe](https://tryhackme.com/room/neighbour)

## Enunciat

Fes un cop d'ull al nostre nou servei al núvol, Authentication Anywhere: inicia la sessió des d'on vulguis! Els usuaris poden introduir el seu nom d'usuari i contrasenya, per a un procés d'inici de sessió totalment segur! Definitivament no podries trobar cap secret que altres persones tinguin al seu perfil, oi?

Accedeix a aquest repte desplegant tant la màquina vulnerable prement el botó verd "Start Lab Machine" que hi ha dins d'aquesta tasca, com el TryHackMe AttackBox prement el botó "Iniciar AttackBox" situat a la part superior dreta de la pàgina.

Ves a la URL següent utilitzant l'AttackBox: http://MACHINE_IP

Fes un cop d'ull a contingut similar a TryHackMe:

- IDOR

## Reconeixement

Per a aquesta màquina el que farem serà accedir a la web de la qual ens parlen:

![Captura de com es veu la pàgina web d'aquesta màquina](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_web-Neighbour-THM.png)

D'acord, veiem que ens demanen que iniciem sessió en aquesta pàgina web, però no sabem ni l'usuari ni la contrasenya, tot i que ens diu que si no en tenim podem fer servir el compte de guest:

![Pista que ens dona la mateixa pàgina](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_Pista-Neighbour-THM.png)

La pàgina ens recomana fer servir el compte de convidat, però no diu la contrasenya. Anem a mirar el codi font amb Ctrl+U, per si hi ha alguna pista.

![Codi de la pàgina web](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_codigo_web_Neighbour-THM.png)

Ara que hem llegit el codi, podem veure que té aquesta secció on diu que podem iniciar sessió fent servir com a usuari i contrasenya unes credencials que han quedat exposades al codi.

![Captura de la secció on surt que podem iniciar sessió com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_codigo_guest-Neighbour-THM.png)

Amb l'usuari i la contrasenya, el que farem serà tornar al login i posar-hi aquestes dades.

![Login amb el compte de convidat](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Login_guest_Neighbour-THM.png)

Perfecte, com podem veure, hem iniciat sessió com a guest, i amb això podem veure diverses coses. Per començar, el missatge: "Hola, convidat. Benvingut al nostre lloc. Intenta no mirar el perfil del teu veí."

![Login reeixit com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_dentro_guest_Neighbour-THM.png)

I ja a dins, ens podem fixar en la manera que utilitzen per identificar l'usuari guest:

![URL de la pàgina un cop iniciada la sessió com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_Neighbour-THM.png)

Amb això ja podem passar a la identificació de vulnerabilitats.

## Identificació de la vulnerabilitat

La principal cosa que m'ha cridat l'atenció d'aquesta màquina ha estat la URL de l'usuari guest un cop hem iniciat sessió, el final, on posa guest.

![Identificador d'usuari](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Identificador_usuario_Neighbour-THM.png)

Amb això sospito que es tracta d'una vulnerabilitat IDOR, amb la qual podríem saltar a l'usuari admin simplement canviant la URL de manera que, en comptes de veure la pàgina d'user, es vegi la pàgina d'admin.

## Explotació

Per a l'explotació del que sospito que és aquesta IDOR simplement hem de canviar la URL perquè, en comptes de tenir user=guest, sigui user=admin.

---

#### Què és un IDOR?

IDOR (Insecure Direct Object Reference) és una vulnerabilitat de control d'accés que es produeix quan una aplicació fa servir un identificador controlat per l'usuari (com `user=guest` a la URL) per accedir a una pàgina o a un altre recurs, sense comprovar si aquest usuari té permís per veure'l. En modificar l'identificador, podem accedir a dades d'altres usuaris.

---

Així doncs, canviem això a la nostra URL:

![Canviant la URL perquè en lloc de guest posi admin per a l'IDOR](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_IDOR-Neighbour-THM.png)

I amb això premem Enter per anar a la pàgina:

![Captura de la pàgina d'admin](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

Perfecte! Amb això hem pogut entrar a la pàgina de l'usuari administrador i hem aconseguit la flag.

## Obtenció de la flag

I amb això ja tenim la flag d'aquesta màquina, que hem obtingut gràcies a un IDOR en aquesta pàgina web.

![Flag d'aquest nivell](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

## Gràcies per llegir.