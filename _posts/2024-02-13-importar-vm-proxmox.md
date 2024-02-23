---
layout: post
title: Importar maquina virtual VirtualBox o VMWare a Proxmox
subtitle:  Procedimiento para importar una maquina virtual de VirtualBox o VMWare a Proxmox
tags: [VirtualBox, VMWare, Proxmox]
comments: true
youtubeId: mUrRezk23DQ
---

# Importar maquina virtual de VirtualBox o VMWare a Proxmox
En este post trataremos de explicar de forma sencilla como importar una maquina existente en Virtual Box o VMware a Proxmox, a continuación los pasos a seguir para el proceso.

## Exportar maquina virtual de VMWare o VirtualBox
Dependiendo de su necesidad. Debe exportar la maquina virtual a ```.ova``` en este momento no explicaremos como exportar la MV a ```.ova``` ya que está en otros posts.

## ¿Como importar la maquina virtual a Proxmox?
- Primero le cambiamos la extensión al archivo ```.ova``` para extensión ```.zip ```. Dentro tendra varios archivos pero nos intereza el archivo ```.vmdk``` y el archivo ```.ovf```.
- Subimos a nuestro servidor proxmox mediante ssh nuestros archivos ```maquinaVirtual1.vmdk``` y ```maquinaVirtual1.ovf ```, puede ser al directorio ```/tmp```

![Importar VM a Proxmox](/assets/img/importar-vm-proxmox/subirvm.png "Importar VM a Proxmox")

- Entramos a nuestro servidor Proxmox por ssh.

```
~$ ssh root@myhost.com
```

- Luego vamos nos dirigimos a directorio /tmp que es donde copiamos los archivos

```
~$ cd /tmp
```

- Ahora importaremos los orchivos, primeramente el archivo correspondiente con a VM ```.ovf```, luego el archivo de disco duro ```.vmdk```, para nuestra prueba utilizaremos una maquina virtual con pfsense instalado

```
~# qm importovf 210 pfsense.ovf local-lvm
~# qm importdisk 210 pfsense-disk001.vmdk local-lvm
```

210 seria el nuevo identificador de la maquina virtual <br>

- Luego le agregamos el disco duro a la maquina virtual, haciendo doble click sobre el disco

![Agregar HDD a VM Importada](/assets/img/importar-vm-proxmox/vmagregarhdd.png "Agregar HDD a VM en proxmox")

verificar si todo funciona correctamente, si todo funciono correctamente su VM debe encender correctamente

![Iniciar VM en Proxmox](/assets/img/importar-vm-proxmox/probarvmimportada.png "VM Importada iniciando")

¡O si lo prefiere puede ver en youtube los pasos!

{% include youtubePlayer.html id=page.youtubeId %}