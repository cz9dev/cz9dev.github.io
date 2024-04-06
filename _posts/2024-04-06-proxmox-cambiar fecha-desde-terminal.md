---
layout: post
title: Cambiar la fecha de servidor Proxmox desde la terminal comandos utiles
subtitle:  Como podemos cambiar la fecha de proxmox desde la terminal
cover-img: /assets/img/posts/proxmox.png
thumbnail-img: /assets/img/posts/proxmox.png
tags: [Proxmox]
comments: true
---

# Cambiar la fecha del servidor proxmox desde la terminal
~~~
date --set "2014-08-31 09:00"
date --set="23 June 1988 10:00:00"
date --set="10:00:00"
~~~
# Configurar el reloj del hardware a la hora del sistema actual.
```
# hwclock --systohc
```
# Configure la hora del sistema desde el reloj del hardware.
```
# hwclock --hctosys
```