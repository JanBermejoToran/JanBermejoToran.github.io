+++
date = '2026-10-02T01:54:53+02:00'
draft = false
title = 'Enabling Wake-on-LAN on Proxmox servers'
summary = 'How to enable Wake-on-LAN on a Proxmox server'
tags = ["Proxmox", "HomeLab", "tutorial"]
+++


![Post cover image](/images/Proxmox/Wake_up_on_LAN/wol_proxmox.png)


## TL;DR

1. Enable WoL in the BIOS/UEFI (and disable ErP/EuP if needed).
2. Set the NIC to mode `g` with `ethtool -s nic0 wol g` and make it persistent with `post-up` in `/etc/network/interfaces`.
3. Test with `wakeonlan <MAC>` from another machine or from another node of the Proxmox cluster.


## Why?

Since I have several servers and manage them remotely, I need to be able to shut them down when I'm not using them so I don't waste electricity just keeping them on. My goal is to set up WoL (Wake-on-LAN) so I can power the servers off and turn them back on without pressing the physical power button.

## What is WoL?

WoL stands for "Wake-on-LAN". It lets you send a packet over the network to a machine with the instruction to power on, so you can turn on a server or computer that is off. What we do need is for the machine to be connected to the network and to power: if either is missing, it simply won't work. Power is needed for the machine to run at all, and the network is needed to deliver the wake-up packet.

### But what is this wake-up packet?

That packet is a "magic packet": a network frame made of 6 bytes of `FF` followed by the card's MAC address repeated 16 times. It is sent as a broadcast (usually UDP to port 9), and the NIC, which stays powered while the machine is off, recognizes it and turns the machine on.

## Requirements

- Motherboard and network card that support WoL
- Wired Ethernet connection (Wi-Fi is not reliable, although it might be possible)
- A machine on the same local network to send the packet

---
## What is the NIC?

*The NIC is the Network Interface Card*

I'm pointing this out because we'll be using this term throughout the blog and I want it to be clear.

---



## Step 1: enable it in the BIOS/UEFI

If the machine we want to configure meets the requirements above, the next step to enable this feature is in the BIOS.

So let's enter the BIOS

![Photo of the BIOS main screen](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_inicio.jpg)

Once inside, we'll look for the WoL option, but depending on the motherboard manufacturer the name may vary.

It can be one of these:
- "Wake On Lan"
- "Power On By PCI-E"
- "Resume by LAN"

Once we find it, we'll enable it:

![Photo of the BIOS option enabled](/images/Proxmox/Wake_up_on_LAN/Captura_Bios_wol_enable.jpg)

Additionally, you may need to disable "ErP/EuP", which cuts power to the NIC when the computer is turned off.



## Step 2: check the NIC in Proxmox

Now that we've enabled WoL in the BIOS and, if needed, disabled "ErP/EuP", we'll identify the name of our physical interface, which in Proxmox is usually "nic0", "eno1" or "enp3s0"

```bash
ip link
```

![Screenshot of the Proxmox terminal with the ip link command](/images/Proxmox/Wake_up_on_LAN/Captura_shell_proxmox_ip_link.png)
 
Now that we know the name of our physical interface, we'll also note down the MAC address, which we'll need later, install `ethtool` in case it isn't already installed on our Proxmox server, and check the WoL status:

```bash
apt install ethtool
ethtool nic0 | grep Wake-on
```
So we check whether the package is installed.

![ethtool package check](/images/Proxmox/Wake_up_on_LAN/Comprobación_paquete_ethtool.png)

Since it's already installed, let's check whether WoL mode is enabled or disabled:

![Check to see whether it is set to d or g](/images/Proxmox/Wake_up_on_LAN/WoL_activo_d.png)

If `Wake-on` shows `d` (disabled), we need to enable it. The mode we want is `g` (magic packet):

```bash
ethtool -s nic0 wol g
```

![Changing it so it is active](/images/Proxmox/Wake_up_on_LAN/WoL_activo_g.png)

With this, WoL is now active, but only temporarily.

## Step 3: make it persistent

The previous setting is lost on reboot, and obviously I don't want that. In Proxmox the cleanest way is to add it to `/etc/network/interfaces`, inside the physical interface's block. To do that:

```bash
iface nic0 inet manual
    post-up /usr/sbin/ethtool -s nic0 wol g
```

Now we apply the configuration without rebooting:

```bash
ifreload -a
```

You can also reboot the machine. Then we check again with `ethtool nic0 | grep Wake-on`: it should show `g`. If it doesn't, we should review the configuration.

## Step 4: test it


Now comes the most important part: testing it. Let's check that this configuration works as it should. To do so, we simply shut down the server:

```bash
shutdown -h now
```
We can turn the server on from another machine on the network by sending the packet with the NIC's MAC address using this command:

```bash
wakeonlan AA:BB:CC:DD:EE:FF
```

![Screenshot of the command to send the packet](/images/Proxmox/Wake_up_on_LAN/Comando_paquete_envio.png)

To send this packet from Arch Linux, I had to install the tool:

![wakeonlan installation](/images/Proxmox/Wake_up_on_LAN/Instalación_herramienta_wakeonlan.png)

As an alternative, we can use `etherwake`: `etherwake -i eth0 AA:BB:CC:DD:EE:FF`.

And when the card receives the packet, it sends the order to turn the server on.

---


In my case, since I have a cluster with three Proxmox servers, I can test it from another Proxmox server that is already on. For that, we need to set the MAC address of the server we want to wake with WoL.

![Machine MAC address configured in Proxmox so it can detect it and we can use WoL](/images/Proxmox/Wake_up_on_LAN/Mac_configurada_wol_proxmox.png)
 

So, with that configuration in place, we can shut the machine down, and once it's off, I right-click its name and the option appears at the very bottom:


![Proxmox WoL photo](/images/Proxmox/Wake_up_on_LAN/Captura_Wol_proxmox.png)

And if everything is set up correctly, after a while it should show that the machine has turned on.


![Screenshot of the machine powered on](/images/Proxmox/Wake_up_on_LAN/Captura_maquina_encendida.png)




## What about virtual machines?

WoL wakes the Proxmox host, not the VMs. For them to start on their own, enable *Start at boot* on the VMs you want to start automatically.

## Security

The magic packet has no authentication or encryption, so anyone on the LAN can turn this server on. It's not serious by itself, but I won't expose it to the internet. If I want to wake it from outside my network, I use a VPN for security. 

## Common problems

- **Goes back to `d` after a reboot:** the setting isn't persistent. Check that `post-up` is in the physical interface's block (`nic0`, not `vmbr0`) and apply with `ifreload -a`.

- **`Supports Wake-on` doesn't include `g`:** the NIC or its driver doesn't support magic packets. There is no software fix for this.

- **Turns on after a reboot but not after a shutdown:** it's almost always the BIOS "ErP/EuP" cutting power to the NIC. Disable it.

- **The network port light turns off when the server shuts down:** the NIC isn't receiving power, so it can't listen for the packet. Same cause: ErP/EuP or a power-saving option in the BIOS.

- **The packet doesn't arrive:** check that the sending machine is on the same network (no VLAN or different subnet) and that the switch or router isn't blocking broadcast.

- **Wrong MAC:** use the MAC of the physical NIC shown in `ip link`, not that of another interface.

- **The option doesn't appear in Proxmox:** the MAC isn't configured on the node, or there is no other cluster node powered on to send the packet from.


## Conclusion

Four steps: BIOS, `ethtool`, persistence and testing.

It hasn't been complicated if you follow the steps carefully. Any problems that come up depend more on the hardware: the result depends on the compatibility of your motherboard and your NIC.

But with this, we'd already have WoL enabled and persistent on our Proxmox server.