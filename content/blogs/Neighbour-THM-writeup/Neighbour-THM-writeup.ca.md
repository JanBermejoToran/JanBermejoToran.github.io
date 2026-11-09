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

## Enunciat

Fes un cop d'ull al nostre nou servei al núvol, Authentication Anywhere - inicia sessió des de qualsevol lloc que vulguis! Els usuaris poden introduir el seu nom d'usuari i contrasenya, per a un procés d'inici de sessió totalment segur! Definitivament no podries trobar cap secret que altres persones tinguin al seu perfil, oi?


Accedeix a aquest repte desplegant tant la màquina vulnerable prement el botó verd "Start Lab Machine" ubicat dins d'aquesta tasca, com el TryHackMe AttackBox prement el botó "Iniciar AttackBox" ubicat a la part superior dreta de la pàgina.

Ves a la següent URL fent servir l'AttackBox: http://MACHINE_IP


Fes un cop d'ull a contingut similar a TryHackMe:

- IDOR

## Reconeixement

Per a aquesta màquina el que farem serà accedir a la web de la qual ens parlen:

![Captura de com es veu la pàgina web d'aquesta màquina](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_web-Neighbour-THM.png)


Ok, veiem com ens demanen que iniciem sessió en aquesta pàgina web, però no sabem ni l'usuari ni la contrasenya, però veiem que ens diu que si no tenim usuari ni contrasenya podem fer servir el compte de guest:

![Pista que ens dona la mateixa pàgina](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_Pista-Neighbour-THM.png)

Així que ho farem, ja que bàsicament fent servir ctrl+u podem veure el codi de la pàgina:

![Codi de la pàgina web](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_codigo_web_Neighbour-THM.png)

Ara que ens hem llegit el codi, podem veure que té aquesta secció d'aquí que diu que podem iniciar sessió fent servir com a usuari i contrasenya: guest

![Captura de la secció on surt que podem iniciar com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_remarcado_codigo_guest-Neighbour-THM.png)

Amb l'usuari i la contrasenya, el que farem serà tornar al login i posarem aquestes dades.

![Login amb guest com a usuari i contrasenya guest:guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Login_guest_Neighbour-THM.png)

Perfecte, com podem veure, hem iniciat sessió com a guest, i amb això podem veure diverses coses, per començar el missatge, que en català significa: "Hola, convidat. Benvingut al nostre lloc. Intenta no mirar el perfil del teu veí."

![Login reeixit com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_dentro_guest_Neighbour-THM.png)

I estant ja a dins, ens podem fixar en la forma que utilitzen per estar dins de l'usuari guest:

![URL de la pàgina estant ja amb la sessió iniciada com a guest](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_Neighbour-THM.png)

Amb això podem passar ja a la identificació de vulnerabilitats


## Identificació de la vulnerabilitat

La principal cosa que m'ha cridat l'atenció d'aquesta màquina ha estat la URL de l'usuari guest quan ja hem iniciat sessió, el final, on diu guest.

![Identificador d'usuari](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Identificador_usuario_Neighbour-THM.png)

Amb això sospito que es tracta d'una vulnerabilitat IDOR, amb la qual podríem saltar a l'usuari admin simplement canviant la URL de tal manera que en lloc de veure la pàgina de user, es vegi la pàgina d'admin.


## Explotació

Per a l'explotació del que sospito que és aquesta IDOR simplement hem de canviar la URL perquè en lloc de tenir user=guest sigui user=admin.

---

#### Què és un IDOR?

IDOR (Insecure Direct Object Reference) és una vulnerabilitat de control d'accés que passa quan una aplicació fa servir un identificador controlat per l'usuari (com `user=guest` a la URL) per accedir a una pàgina o altre recurs, sense comprovar si aquest usuari té permís per veure'l. En modificar l'identificador, podem accedir a dades d'altres usuaris.

---

Així que canviem això a la nostra URL:

![Canviant la URL perquè en lloc de guest posi admin per a l'IDOR](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/url_IDOR-Neighbour-THM.png)

I amb això premem enter per anar a la pàgina:

![Captura de la pàgina d'admin](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)

Perfecte! Amb això hem pogut entrar a la pàgina de l'usuari administrador i hem aconseguit la flag.

## Obtenció de la flag

I amb això ja tenim la flag d'aquesta màquina, la qual hem obtingut gràcies a una IDOR en aquesta pàgina web.

![Flag d'aquest nivell](/images/writeups/TryHackMe_imágenes/Neighbour-THM-imágenes/Captura_flag_Neighbour-THM.png)


## Gràcies per llegir.