+++
date = '2026-10-02T01:54:53+02:00'
draft = false
title = 'Activar Wake-on-LAN en servidores Proxmox'
summary = 'Explicación de cómo activar Wake-on-LAN en un servidor Proxmox'
tags = ["Proxmox", "HomeLab", "tutorial"]
+++


![Foto inicio del post](/images/Proxmox/Wake_up_on_LAN/wol_proxmox.png)


## TL;DR

1. Activa WoL en la BIOS/UEFI (y desactiva ErP/EuP si hace falta).
2. Pon la NIC en modo `g` con `ethtool -s nic0 wol g` y hazlo persistente con `post-up` en `/etc/network/interfaces`.
3. Prueba con `wakeonlan <MAC>` desde otro equipo o desde otro nodo del clúster de Proxmox.


## ¿Por qué?

Al tener varios servidores y administrarlos a distancia, debo poder apagarlos al dejar de usarlos para no necesitar estar gastando tanta luz simplemente teniéndolos encendidos, así que mi objetivo es poner WoL (Wake-on-LAN) para poder apagar los servidores y luego encenderlos de nuevo sin necesitar pulsar el botón de encendido físico.

## ¿Qué es WoL?

El WoL son las siglas de "Wake-on-LAN", que sirve para poder mandar un paquete a esta máquina por la red para poder mandarle la instrucción de encendido, con la finalidad de poder encender un servidor o ordenador que esté apagado, lo que sí que necesitamos es que este ordenador o servidor esté conectado a la red y a la luz, ya que si uno de estos dos no se cumple, simplemente no funcionará, la luz es necesaria para que este ordenador funcione y la red es necesaria para poder mandar el paquete de encendido.

### Pero, ¿qué es este paquete de encendido?

Ese paquete es un "magic packet", es una trama de red con 6 bytes `FF` seguidos de la MAC de la tarjeta repetida 16 veces. Se envía por broadcast (normalmente UDP al puerto 9) y la NIC, que sigue alimentada con el equipo apagado, lo reconoce y enciende la máquina.

## Requisitos

- Placa y tarjeta de red compatibles con WoL
- Conexión por cable Ethernet (por Wi-Fi no es fiable, aunque podría llegar a hacerse)
- Un equipo en la misma red local para enviar el paquete

---
## ¿Qué es la NIC?

*La NIC es la Network Interface Card*

Lo remarco porque en este blog vamos a hablar de este término y quiero que quede claro.

---



## Paso 1: activarlo en la BIOS/UEFI

Si el equipo que queremos configurar cumple con los requisitos anteriores, el siguiente paso que deberemos hacer para poder activar esta función está en la BIOS.

Así que entraremos en la BIOS

![Foto de la pantalla principal de la BIOS](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_inicio.jpg)

Y una vez dentro vamos a buscar la opción de WoL, pero dependiendo de qué fabricante sea la placa puede variar el nombre que le ponen.

Puede ser uno de estos:
- "Wake On Lan"
- "Power On By PCI-E"
- "Resume by LAN"

Una vez la encontremos, lo que vamos a hacer va a ser activarla:

![Foto de la opción en la BIOS activada](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_wol_enable.jpg)

Además, podría ser que necesitemos desactivar el "ErP/EuP", que se encarga de cortar la alimentación de la NIC al apagar el ordenador



## Paso 2: comprobar la NIC en Proxmox

Ahora que ya hemos activado la opción de WoL en la BIOS, y en caso de necesitarlo también desactivado el "ErP/EuP", lo que vamos a hacer va a ser identificar el nombre de nuestra interfaz física, que en Proxmox suele ser "nic0", "eno1" o "enp3s0"

```tmux
ip link
```

![Captura de la terminal de Proxmox con el comando ip link](/images/Proxmox/Wake_up_on_LAN/Captura_shell_proxmox_ip_link.png)
 
Ahora que ya sabemos el nombre de nuestra interfaz física, vamos a además apuntarnos la dirección MAC, que nos hará falta más tarde y vamos a instalar `ethtool` en caso de no tenerlo ya instalado en nuestro servidor Proxmox y consultaremos el estado de WoL:

```tmux
apt install ethtool
ethtool nic0 | grep Wake-on
```
Así que comprobamos si tenemos instalado el paquete.

![Comprobación de paquete ethtool](/images/Proxmox/Wake_up_on_LAN/Comprobación_paquete_ethtool.png)

Como ya lo tenemos instalado vamos a probar para ver si tenemos activado o desactivado el modo WoL:

![Comprobación para ver si está en d o en g](/images/Proxmox/Wake_up_on_LAN/WoL_activo_d.png)

Si `Wake-on` aparece como `d` (disabled), hay que activarlo. El modo que nos interesa es `g` (magic packet):

```tmux
ethtool -s nic0 wol g
```

![Cambiando para que esté activo](/images/Proxmox/Wake_up_on_LAN/WoL_activo_g.png)

Con esto el WoL queda activo, pero solo de forma temporal.

## Paso 3: hacerlo persistente

El ajuste anterior se pierde al reiniciar, y evidentemente no quiero eso. En Proxmox lo más limpio es añadirlo en `/etc/network/interfaces`, dentro del bloque de la interfaz física, para eso, lo que vamos a hacer es:

```tmux
iface nic0 inet manual
    post-up /usr/sbin/ethtool -s nic0 wol g
```

Ahora aplicamos la configuración sin reiniciar con:

```tmux
ifreload -a
```

También puedes reiniciar la máquina. Después volvemos a comprobar con `ethtool nic0 | grep Wake-on`: debe mostrar `g`, si no sale `g`, deberíamos revisar la configuración.

## Paso 4: probarlo


Ahora queda lo más importante, probarlo, así que vamos a probar si esta configuración funciona como debería, para ello, simplemente apagamos el servidor:

```tmux
shutdown -h now
```
Podemos encender el servidor desde otro equipo de la red, enviando el paquete con la MAC de la NIC con este comando:

```tmux
wakeonlan AA:BB:CC:DD:EE:FF
```

![Captura del comando para enviar el paquete](/images/Proxmox/Wake_up_on_LAN/Comando_paquete_envio.png)

Para enviar este paquete desde un Arch Linux me ha sido necesario instalar la herramienta:

![instalación wakeonlan](/images/Proxmox/Wake_up_on_LAN/Instalación_herramienta_wakeonlan.png)

Como alternativa, podemos usar `etherwake`: `etherwake -i eth0 AA:BB:CC:DD:EE:FF`.

Y cuando la tarjeta reciba el paquete, se mandará la orden para encender el servidor.

---


En mi caso, como tengo un clúster con tres servidores de Proxmox, puedo probarlo desde otro servidor Proxmox que ya esté encendido. Para ello necesitamos configurar la MAC del servidor que queremos que se encienda con WoL.

![MAC de la máquina configurada en Proxmox para que la detecte y podamos usar WoL](/images/Proxmox/Wake_up_on_LAN/Mac_configurada_wol_proxmox.png)
 

Así que teniendo ya esa configuración podemos apagar la máquina, y una vez está apagada, le doy botón derecho encima de su nombre, y me sale la opción debajo de todo:


![foto WoL de Proxmox](/images/Proxmox/Wake_up_on_LAN/Captura_Wol_proxmox.png)

Y si está bien hecho, después de un rato, debería salir como la máquina se ha encendido.


![Captura máquina encendida](/images/Proxmox/Wake_up_on_LAN/Captura_maquina_encendida.png)




## ¿Y las máquinas virtuales?

WoL despierta el host Proxmox, no las VMs. Para que arranquen solas, activa *Start at boot* en las VM que quieras arrancar.

## Seguridad

El magic packet no tiene autenticación ni cifrado, así que cualquiera en la LAN puede encender este servidor. No es grave por sí solo, pero yo no lo voy a exponer a internet. Si quiero encenderlo desde fuera de mi red uso una VPN por seguridad. 

## Problemas habituales

- **Vuelve a `d` tras reiniciar:** el ajuste no es persistente. Revisa que el `post-up` esté en el bloque de la interfaz física (`nic0`, no `vmbr0`) y aplica con `ifreload -a`.

- **`Supports Wake-on` no incluye `g`:** la NIC o su driver no soportan magic packet. Aquí no hay solución por software.

- **Enciende tras reiniciar pero no tras apagar:** casi siempre es el "ErP/EuP" de la BIOS cortando la alimentación de la NIC. Desactívalo.

- **La luz del puerto de red se apaga al apagar el servidor:** la NIC no recibe corriente, así que no puede escuchar el paquete. Mismo origen: ErP/EuP o ahorro de energía en la BIOS.

- **El paquete no llega:** comprueba que el equipo emisor está en la misma red (sin VLAN ni subred distinta) y que el switch o router no bloquea el broadcast.

- **MAC equivocada:** usa la de la NIC física que ves en `ip link`, no la de otra interfaz.

- **No aparece la opción en Proxmox:** falta configurar la MAC en el nodo, o no hay otro nodo del clúster encendido desde el que mandar el paquete.


## Conclusión

Cuatro pasos: BIOS, `ethtool`, persistencia y prueba.

No ha sido algo complicado si se siguen bien los pasos. Los problemas que pueden surgir dependen más del hardware: el resultado depende de la compatibilidad de tu placa y tu NIC.

Pero con esto ya tendríamos el WoL activo en nuestro servidor Proxmox con persistencia.