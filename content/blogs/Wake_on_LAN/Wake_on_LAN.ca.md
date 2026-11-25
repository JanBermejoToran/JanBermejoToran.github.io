+++
date = '2026-10-02T01:54:53+02:00'
draft = false
title = 'Activar Wake-on-LAN en servidors Proxmox'
summary = 'Explicació de com activar Wake-on-LAN en un servidor Proxmox'
tags = ["Proxmox", "HomeLab", "tutorial"]
+++


![Imatge d'inici del post](/images/Proxmox/Wake_up_on_LAN/wol_proxmox.png)


## TL;DR

1. Activa el WoL a la BIOS/UEFI (i desactiva l'ErP/EuP si cal).
2. Posa la NIC en mode `g` amb `ethtool -s nic0 wol g` i fes-ho persistent amb `post-up` a `/etc/network/interfaces`.
3. Prova-ho amb `wakeonlan <MAC>` des d'un altre equip o des d'un altre node del clúster de Proxmox.


## Per què?

Com que tinc diversos servidors i els administro a distància, he de poder apagar-los quan deixo d'utilitzar-los per no gastar tanta llum només tenint-los encesos. Així que el meu objectiu és configurar el WoL (Wake-on-LAN) per poder apagar els servidors i tornar-los a encendre sense haver de prémer el botó d'encesa físic.

## Què és el WoL?

El WoL són les sigles de "Wake-on-LAN". Serveix per enviar un paquet a una màquina per la xarxa amb la instrucció d'encendre's, de manera que es pugui encendre un servidor o ordinador que estigui apagat. El que sí que necessitem és que l'equip estigui connectat a la xarxa i al corrent elèctric: si una de les dues coses falla, simplement no funcionarà. El corrent és necessari perquè l'ordinador pugui funcionar, i la xarxa, per poder enviar el paquet d'encesa.

### Però, què és aquest paquet d'encesa?

Aquest paquet és un "magic packet": una trama de xarxa amb 6 bytes `FF` seguits de la MAC de la targeta repetida 16 vegades. S'envia per broadcast (normalment UDP al port 9) i la NIC, que continua alimentada amb l'equip apagat, el reconeix i encén la màquina.

## Requisits

- Placa i targeta de xarxa compatibles amb WoL
- Connexió per cable Ethernet (per Wi-Fi no és fiable, tot i que es podria arribar a fer)
- Un equip a la mateixa xarxa local per enviar el paquet

---
## Què és la NIC?

*La NIC és la Network Interface Card*

Ho remarco perquè en aquest blog parlarem d'aquest terme i vull que quedi clar.

---



## Pas 1: activar-ho a la BIOS/UEFI

Si l'equip que volem configurar compleix els requisits anteriors, el següent pas per poder activar aquesta funció és a la BIOS.

Així que entrarem a la BIOS

![Foto de la pantalla principal de la BIOS](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_inicio.jpg)

I un cop a dins buscarem l'opció de WoL, però segons el fabricant de la placa el nom pot variar.

Pot ser un d'aquests:
- "Wake On Lan"
- "Power On By PCI-E"
- "Resume by LAN"

Un cop la trobem, l'activarem:

![Foto de l'opció a la BIOS activada](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_wol_enable.jpg)

A més, podria ser que haguem de desactivar l'"ErP/EuP", que s'encarrega de tallar l'alimentació de la NIC quan s'apaga l'ordinador.



## Pas 2: comprovar la NIC a Proxmox

Ara que ja hem activat l'opció de WoL a la BIOS i, si ha calgut, també hem desactivat l'"ErP/EuP", identificarem el nom de la nostra interfície física, que a Proxmox sol ser "nic0", "eno1" o "enp3s0"

```bash
ip link
```

![Captura del terminal de Proxmox amb l'ordre ip link](/images/Proxmox/Wake_up_on_LAN/Captura_shell_proxmox_ip_link.png)
 
Ara que ja sabem el nom de la nostra interfície física, ens apuntarem també l'adreça MAC, que ens farà falta més endavant, instal·larem `ethtool` en cas de no tenir-lo ja instal·lat al nostre servidor Proxmox i consultarem l'estat del WoL:

```bash
apt install ethtool
ethtool nic0 | grep Wake-on
```
Així que comprovem si tenim instal·lat el paquet.

![Comprovació del paquet ethtool](/images/Proxmox/Wake_up_on_LAN/Comprobación_paquete_ethtool.png)

Com que ja el tenim instal·lat, comprovarem si el mode WoL està activat o desactivat:

![Comprovació per veure si està en d o en g](/images/Proxmox/Wake_up_on_LAN/WoL_activo_d.png)

Si `Wake-on` apareix com a `d` (disabled), cal activar-lo. El mode que ens interessa és `g` (magic packet):

```bash
ethtool -s nic0 wol g
```

![Canviant-ho perquè estigui actiu](/images/Proxmox/Wake_up_on_LAN/WoL_activo_g.png)

Amb això el WoL queda actiu, però només de manera temporal.

## Pas 3: fer-ho persistent

L'ajust anterior es perd en reiniciar, i evidentment no ho vull. A Proxmox el més net és afegir-ho a `/etc/network/interfaces`, dins del bloc de la interfície física. Per fer-ho:

```bash
iface nic0 inet manual
    post-up /usr/sbin/ethtool -s nic0 wol g
```

Ara apliquem la configuració sense reiniciar amb:

```bash
ifreload -a
```

També pots reiniciar la màquina. Després tornem a comprovar amb `ethtool nic0 | grep Wake-on`: ha de mostrar `g`; si no surt `g`, hauríem de revisar la configuració.

## Pas 4: provar-ho


Ara toca el més important, provar-ho, així que comprovarem si aquesta configuració funciona com ha de funcionar. Per fer-ho, simplement apaguem el servidor:

```bash
shutdown -h now
```
Podem encendre el servidor des d'un altre equip de la xarxa, enviant el paquet amb la MAC de la NIC amb aquesta ordre:

```bash
wakeonlan AA:BB:CC:DD:EE:FF
```

![Captura de l'ordre per enviar el paquet](/images/Proxmox/Wake_up_on_LAN/Comando_paquete_envio.png)

Per enviar aquest paquet des d'un Arch Linux m'ha calgut instal·lar l'eina:

![instal·lació de wakeonlan](/images/Proxmox/Wake_up_on_LAN/Instalación_herramienta_wakeonlan.png)

Com a alternativa, podem fer servir `etherwake`: `etherwake -i eth0 AA:BB:CC:DD:EE:FF`.

I quan la targeta rebi el paquet, s'enviarà l'ordre per encendre el servidor.

---


En el meu cas, com que tinc un clúster amb tres servidors de Proxmox, puc provar-ho des d'un altre servidor Proxmox que ja estigui encès. Per això hem de configurar la MAC del servidor que volem que s'encengui amb WoL.

![MAC de la màquina configurada a Proxmox perquè la detecti i puguem fer servir WoL](/images/Proxmox/Wake_up_on_LAN/Mac_configurada_wol_proxmox.png)
 

Així que, amb aquesta configuració feta, podem apagar la màquina, i un cop està apagada, hi faig clic dret a sobre del seu nom i em surt l'opció a baix de tot:


![foto WoL de Proxmox](/images/Proxmox/Wake_up_on_LAN/Captura_Wol_proxmox.png)

I si està ben fet, al cap d'una estona hauria de sortir que la màquina s'ha encès.


![Captura de la màquina encesa](/images/Proxmox/Wake_up_on_LAN/Captura_maquina_encendida.png)




## I les màquines virtuals?

El WoL desperta el host Proxmox, no les VMs. Perquè arrenquin soles, activa *Start at boot* a les VM que vulguis arrencar.

## Seguretat

El magic packet no té autenticació ni xifratge, així que qualsevol a la LAN pot encendre aquest servidor. No és greu per si sol, però jo no l'exposaré a internet. Si vull encendre'l des de fora de la meva xarxa, faig servir una VPN per seguretat. 

## Problemes habituals

- **Torna a `d` després de reiniciar:** l'ajust no és persistent. Revisa que el `post-up` sigui al bloc de la interfície física (`nic0`, no `vmbr0`) i aplica amb `ifreload -a`.

- **`Supports Wake-on` no inclou `g`:** la NIC o el seu driver no suporten el magic packet. Aquí no hi ha solució per programari.

- **S'encén després de reiniciar però no després d'apagar:** gairebé sempre és l'"ErP/EuP" de la BIOS tallant l'alimentació de la NIC. Desactiva'l.

- **La llum del port de xarxa s'apaga en apagar el servidor:** la NIC no rep corrent, així que no pot escoltar el paquet. Mateix origen: ErP/EuP o estalvi d'energia a la BIOS.

- **El paquet no arriba:** comprova que l'equip emissor és a la mateixa xarxa (sense VLAN ni subxarxa diferent) i que el switch o el router no bloqueja el broadcast.

- **MAC equivocada:** fes servir la de la NIC física que veus a `ip link`, no la d'una altra interfície.

- **No apareix l'opció a Proxmox:** falta configurar la MAC al node, o no hi ha cap altre node del clúster encès des del qual enviar el paquet.


## Conclusió

Quatre passos: BIOS, `ethtool`, persistència i prova.

No ha estat res complicat si se segueixen bé els passos. Els problemes que poden sorgir depenen més del maquinari: el resultat depèn de la compatibilitat de la teva placa i la teva NIC.

Però amb això ja tindríem el WoL actiu al nostre servidor Proxmox amb persistència.