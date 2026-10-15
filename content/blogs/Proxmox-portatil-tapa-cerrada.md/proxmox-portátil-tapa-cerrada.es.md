+++
date = '2026-09-12T19:48:54+02:00'
draft = false
title = 'Proxmox Portátil Tapa Cerrada'
summary = "write-up de cómo mantener Proxmox encendido en un portátil aunque tenga la tapa cerrada"
tags = ["Proxmox", "HomeLab", "tutorial", "Laptop"]
+++


### Cómo evitar que Proxmox se suspenda al cerrar la tapa de un portátil

![Imagen Visual Tapa Portátil](/images/Proxmox/Tapa_cerrada/proxmox_tapa_cerrada_foto_titulo.png)

En mi caso, uno de mis servidores **Proxmox** lo tengo alojado en un **portátil** por temas de consumo energético.

Si al igual que yo utilizas un portátil como servidor con Proxmox VE, puedes encontrarte con un problema bastante común: al cerrar la tapa del portátil, el sistema entra en suspensión (o se apaga) y, como consecuencia, las máquinas virtuales, contenedores y servicios dejan de estar disponibles.

Por defecto, Linux puede interpretar el cierre de la tapa como una orden para suspender el equipo. En un servidor esto no suele ser lo que queremos.

Así que en este post voy a enseñar cómo configurar Proxmox para que cerrar la tapa del portátil no haga absolutamente nada, permitiendo mantener el servidor funcionando aunque la pantalla esté cerrada.

### El Problema

En mi caso tengo un portátil ejecutando Proxmox VE como servidor.

Mientras la pantalla esté abierta, todo funciona bien. Sin embargo, al cerrar la tapa, el equipo entra en suspensión y el servidor deja de responder.

Esto pasa porque Proxmox está basado en Debian, y utiliza systemd para gestionar determinados eventos relacionados con la alimentación y el cierre de la tapa.

La configuración relevante se encuentra en:

```tmux
/etc/systemd/logind.conf
```

Así que lo primero que vamos a hacer es comprobar la configuración actual


Lo primero es comprobar cómo está configurado actualmente systemd-logind.

Así que vamos a entrar en la shell de Proxmox:


![Imagen Proxmox](/images/Proxmox/captura_shell_proxmox.png)


Desde una consola de Proxmox o por SSH ejecutamos:

```tmux
cat /etc/systemd/logind.conf | grep -E 'HandleLidSwitch|HandleLidSwitchExternalPower'
```

Si aparecen líneas como estas:

```tmux
#HandleLidSwitch=suspend
#HandleLidSwitchExternalPower=suspend
#HandleLidSwitchDocked=ignore
```

Significa que las opciones están comentadas y se está utilizando el comportamiento predeterminado de systemd-logind.

En muchos sistemas, ese comportamiento consiste en suspender el equipo cuando se cierra la tapa.

Así que vamos a modificar logind.conf

Editamos el archivo de configuración:

```tmux
nano /etc/systemd/logind.conf
```
![Imagen con las líneas descomentadas](/images/Proxmox/Tapa_cerrada/Captura_shell_editar_archivo.png)


Buscamos estas líneas:
```tmux
#HandleLidSwitch=suspend
#HandleLidSwitchExternalPower=suspend
#HandleLidSwitchDocked=ignore
```

![Imagen con las líneas comentadas](/images/Proxmox/Tapa_cerrada/Captura_shell_server_lineas_comentadas.png)


Y las modificamos para que queden así:
```tmux
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
![Imagen con las líneas descomentadas](/images/Proxmox/Tapa_cerrada/Captura_shell_sin_comentar.png)

La clave aquí es:
```tmux
ignore
```

Con esta configuración estamos indicando a systemd-logind que **ignore** el evento de que sea al cerrar la tapa.

¿Qué significa cada opción?
```tmux
HandleLidSwitch=ignore
```
Indica qué hacer cuando se cierra la tapa normalmente.
```tmux
HandleLidSwitchExternalPower=ignore
```
Hace lo mismo cuando el portátil está conectado a la alimentación, vamos, cuando está cargando.

```tmux
HandleLidSwitchDocked=ignore
```

Evita que el comportamiento cambie cuando el sistema considera que está conectado a una estación de acoplamiento o configuración similar.

Para un portátil utilizado como servidor, establecer las tres opciones en ignore nos deja cerrar la pantalla sin suspender el equipo.

Aplicar los cambios

Después de guardar el archivo, podemos reiniciar el servicio:

```tmux
systemctl restart systemd-logind
```
![Imagen con las líneas descomentadas](/images/Proxmox/Tapa_cerrada/Captura_shell_reinicio_systemd-login.png)

Aunque también es podríamos reiniciar el servidor para asegurarnos de que toda la configuración se carga desde cero:

```tmux
reboot
```

En mi caso no pasa nada porque simplemente se reinicia el equipo, pero si alguien lo hace por SSH debe tener en cuenta de que al reiniciar lo expulsará de la máquina.

Lo ideal sería al reiniciar, comprobar que la configuración se ha aplicado

Podemos verificar que las opciones ya no están comentadas ejecutando:
```tmux
cat /etc/systemd/logind.conf | grep HandleLidSwitch 
```
![Imagen de la comprobación](/images/Proxmox/Tapa_cerrada/Captura_shell_comprobación.png)

La salida esperada es:
```tmux
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Es importante que revisemos que no aparezca el # al principio de las líneas.

Por ejemplo:
```tmux
#HandleLidSwitch=ignore
```

No estaría activo.

Mientras que:
```tmux
HandleLidSwitch=ignore
```

Sí estaría activo.

### Resultado

Una vez puesta esta configuración, podemos cerrar la tapa del portátil sin que systemd-logind ordene suspender el sistema.

Después de aplicar los cambios, podemos cerrar la tapa tranquilamente y dejar que Proxmox continúe funcionando como servidor con los servicios o lo que tengamos aún activos.

---

#### Importante:
*Importante*: esta configuración evita la acción asociada al cierre de la tapa, pero no impide que el sistema pueda suspenderse por otras causas o por otras configuraciones de gestión de energía. Si el servidor sigue suspendiéndose, habría que revisar también las políticas de systemd, ACPI y la configuración de energía del propio hardware, pero claro, eso depende ya de cada dispositivo, así que esto es lo que me ha funcionado a mí.