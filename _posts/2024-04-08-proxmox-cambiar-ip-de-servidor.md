---
layout: post
title: Cambiar el Ip de un Servidor Proxmox
subtitle:  ¿Cómo podemos cambiar el ip de nuestro servidor proxmox?
## cover-img: /assets/img/posts/proxmox.png
thumbnail-img: /assets/img/posts/proxmox.png
tags: [Proxmox]
comments: true
---

En ocasiones hemos tenido la necesidad de cambiarle la dirección a nuestro servidor Proxmox (PVE). Existen 2 formas rápidas de cambiarle la IP a PVE.

1. A través de la terminal web del mismo servidor
2. A través de la consola o terminal de acceso ssh al servidor

# Pasos para cambiar el IP de PVE
1. Debemos conectar el cable a la interfaz de red que haya sido configurada para la LAN
2. Poner nuestra PC en el mismo rango de IP que el servidor PVE
Aquí es donde cambia si es por una u otra vía
## Por SSH
3. Abrimos la terminal de linux o el puty si estamos en Windows
4. Nos conectamos al servidor
5. Editamos el archivo hosts
```
root@pve:~# nano /etc/hosts
```
6. Cambiamos el IP por el que deseemos, en este momento en la linea 2 donde esta escrito el IP que teniamos ej: 192.168.1.1 ponemos el nuevo IP
```
127.0.0.1 localhost.localdomain localhost
192.168.1.1 pve.myserver.cu pve

# The following lines are desirable for IPv6 capable hosts

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```
7. Una ves realizado el cambio apretamos Ctrl+X y guardamos los cambios
8. Reiniciamos nuestro servidor PVE
```
root@pve:~# reboot
```

Con estos pasos ya deberíamos tener a nuestro servidor PVE en el nuevo rango