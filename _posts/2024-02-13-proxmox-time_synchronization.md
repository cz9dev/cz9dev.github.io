---
layout: post
title: Sincronizar hora de Proxmox contra Servidor NTP local
subtitle:  Como sincronizar nuestro servidor proxmox contra un servidor ntp local
cover-img: /assets/img/posts/proxmox.png
thumbnail-img: /assets/img//posts/proxmox.png
tags: [proxmox]
comments: true
---

# Sincronizar Proxmox contra servidor NTP local
En ocasiones necesitamos sincronizar nuestro servidor Proxmox con un servidor NTP (Servidor de hora) que hayamos instalado en nuestra red local. Para lograrlo tenemos dos opciones, a continuación las mostramos
## Usando chrony
Editamos el archivo /etc/chrony/chrony.conf y en la linea donde aparece pool al inicio la comentamos y escribimos nuestra configuración
```
 Pool ntp.localserver.cu iburst
```
Reiniciamos chrony
```
System restart chronyd
```
Chequeamos nuestra nueva configuración
```
journalctl --since -1h -u chrony
```
## Utilizando systemd-timesyncd:
Editamos el archivo /etc/systemd/timesyncd.conf:
```
[Time]
NTP=ntp1.localserver.cu ntp2.localserver.cu ntp3.localserver.cu ntp4.localserver.cu
```
Luego reiniciamos el servicio synchronization
```
systemctl restart systemd-timesyncd
```
Y verificamos nuestra configuración con:
```
journalctl --since -1h -u systemd-timesyncd

...
Oct 07 14:58:36 node1 systemd[1]: Stopping Network Time Synchronization...
Oct 07 14:58:36 node1 systemd[1]: Starting Network Time Synchronization...
Oct 07 14:58:36 node1 systemd[1]: Started Network Time Synchronization.
Oct 07 14:58:36 node1 systemd-timesyncd[13514]: Using NTP server 10.0.0.1:123 (ntp1.localserver.cu).
Oct 07 14:58:36 node1 systemd-timesyncd[13514]: interval/delta/delay/jitter/drift 64s/-0.002s/0.020s/0.000s/-31ppm
...
```
De esta forma quedara sincronizado nuestro servidor Proxmox contra nuestro servidor NTP local