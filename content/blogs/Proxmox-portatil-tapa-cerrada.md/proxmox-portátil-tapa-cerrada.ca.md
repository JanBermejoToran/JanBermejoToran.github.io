+++
date = '2026-09-12T19:48:54+02:00'
draft = false
title = 'Proxmox amb la tapa del portàtil tancada'
summary = "Guia per mantenir Proxmox en funcionament en un portàtil amb la tapa tancada"
tags = ["Proxmox", "HomeLab", "tutorial", "Laptop"]
+++


### Com evitar que Proxmox se suspengui en tancar la tapa d'un portàtil

![Imatge de la tapa del portàtil tancada](/images/Proxmox/Tapa_cerrada/proxmox_tapa_cerrada_foto_titulo.png)

En el meu cas, tinc un dels meus servidors **Proxmox** en un **portàtil** per qüestions de consum energètic.

Si, com jo, fas servir un portàtil com a servidor amb Proxmox VE, et pots trobar amb un problema força habitual: en tancar la tapa del portàtil, el sistema se suspèn (o s'apaga) i, com a conseqüència, les màquines virtuals, els contenidors i els serveis deixen d'estar disponibles.

Per defecte, Linux pot interpretar que tancar la tapa és una ordre per suspendre l'equip. En un servidor, normalment no és això el que volem.

En aquest article t'ensenyaré a configurar Proxmox perquè en tancar la tapa del portàtil no passi res i el servidor continuï funcionant encara que la pantalla estigui tancada.

### El problema

En el meu cas, tinc un portàtil amb Proxmox VE que fa de servidor.

Amb la pantalla oberta, tot funciona bé. En canvi, en tancar la tapa, l'equip se suspèn i el servidor deixa de respondre.

Això passa perquè Proxmox es basa en Debian i utilitza systemd per gestionar determinats esdeveniments relacionats amb l'alimentació i el tancament de la tapa.

La configuració rellevant es troba a:

```tmux
/etc/systemd/logind.conf
```

Per tant, el primer que farem és comprovar la configuració actual.


Primer comprovarem com està configurat actualment systemd-logind.

Entrem a l'intèrpret d'ordres de Proxmox:


![Consola de Proxmox](/images/Proxmox/captura_shell_proxmox.png)


Des d'una consola de Proxmox o per SSH, executem:

```tmux
cat /etc/systemd/logind.conf | grep -E 'HandleLidSwitch|HandleLidSwitchExternalPower'
```

Si apareixen línies com aquestes:

```tmux
#HandleLidSwitch=suspend
#HandleLidSwitchExternalPower=suspend
#HandleLidSwitchDocked=ignore
```

Vol dir que les opcions estan comentades i que s'utilitza el comportament predeterminat de systemd-logind.

En molts sistemes, aquest comportament consisteix a suspendre l'equip quan se'n tanca la tapa.

Ara modificarem logind.conf.

Editem el fitxer de configuració:

```tmux
nano /etc/systemd/logind.conf
```
![Imatge amb les línies descomentades](/images/Proxmox/Tapa_cerrada/Captura_shell_editar_archivo.png)


Busquem aquestes línies:
```tmux
#HandleLidSwitch=suspend
#HandleLidSwitchExternalPower=suspend
#HandleLidSwitchDocked=ignore
```

![Imatge amb les línies comentades](/images/Proxmox/Tapa_cerrada/Captura_shell_server_lineas_comentadas.png)


I les modifiquem perquè quedin així:
```tmux
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
![Imatge amb les línies descomentades](/images/Proxmox/Tapa_cerrada/Captura_shell_sin_comentar.png)

La clau és:
```tmux
ignore
```

Amb aquesta configuració indiquem a systemd-logind que **ignori** l'esdeveniment que es produeix en tancar la tapa.

Què significa cada opció?
```tmux
HandleLidSwitch=ignore
```
Indica què cal fer quan es tanca la tapa normalment.
```tmux
HandleLidSwitchExternalPower=ignore
```
Fa el mateix quan el portàtil està connectat a l'alimentació, és a dir, quan s'està carregant.

```tmux
HandleLidSwitchDocked=ignore
```

Evita que el comportament canviï quan el sistema considera que està connectat a una estació d'acoblament o a una configuració semblant.

Si fem servir un portàtil com a servidor, establir les tres opcions en ignore ens permet tancar la pantalla sense suspendre l'equip.

Aplicar els canvis

Després de desar el fitxer, podem reiniciar el servei:

```tmux
systemctl restart systemd-logind
```
![Imatge amb les línies descomentades](/images/Proxmox/Tapa_cerrada/Captura_shell_reinicio_systemd-login.png)

També podríem reiniciar el servidor per assegurar-nos que tota la configuració es carrega des de zero:

```tmux
reboot
```

En el meu cas no passa res perquè simplement es reinicia l'equip, però si algú ho fa per SSH ha de tenir en compte que el reinici el desconnectarà de la màquina.

L'ideal seria comprovar després del reinici que la configuració s'ha aplicat.

Podem verificar que les opcions ja no estan comentades executant:
```tmux
cat /etc/systemd/logind.conf | grep HandleLidSwitch 
```
![Imatge de la comprovació](/images/Proxmox/Tapa_cerrada/Captura_shell_comprobación.png)

La sortida esperada és:
```tmux
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

És important comprovar que no aparegui el símbol # al principi de les línies.

Per exemple:
```tmux
#HandleLidSwitch=ignore
```

No estaria activa.

En canvi:
```tmux
HandleLidSwitch=ignore
```

Sí que estaria activa.

### Resultat

Un cop aplicada aquesta configuració, podem tancar la tapa del portàtil sense que systemd-logind ordeni suspendre el sistema.

Després d'aplicar els canvis, podem tancar la tapa tranquil·lament i deixar que Proxmox continuï funcionant com a servidor amb els serveis o qualsevol altra cosa que encara tinguem en marxa.

---

#### Important:
*Important*: aquesta configuració evita l'acció associada al tancament de la tapa, però no impedeix que el sistema se suspengui per altres causes o per altres configuracions de gestió de l'energia. Si el servidor continua suspenent-se, també caldrà revisar les polítiques de systemd, ACPI i la configuració d'energia del mateix maquinari. Això depèn de cada dispositiu; aquesta és la solució que m'ha funcionat a mi.
