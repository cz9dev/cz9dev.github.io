---
layout: post
title: Actualizar Zimbra buld Zextras con todos los parches de seguridad - ¡Importantes!
subtitle: ¿Como actualizar Zimbra buid Zextras en pocos pasos?
cover-img: /assets/img/zimbra_path.png
thumbnail-img: /assets/img/zimbra_thumb.png
tags: [ubuntu, zimbra, zextras, tutorial]
comments: true
---

¿Por que no pudimos actualizar a Zimbra 9 como se hacia antes?. Pues como ya debes conocer Zimbra/Synacor deja de dar soporte (crear parches) desde Zimbra 9 Patch 15, sin motivos aparentes. Lo cual es un problema ya que desde entonces, se han lanzado varios patches, puedes mirar [aqui](https://wiki.zimbra.com/wiki/Zimbra_Releases) para ver el ultimo

Bueno sin mas ¿Como actualizamos a la ultima versión con los patches incluidos?

## ¿Como actualizamos Zimbra - Zextras?

- Descargar el ultimo instalador desde [aquí](https://www.zextras.com/zextras-build-based-on-zimbra-official-repository/)
- Descomprime el instalador, y como root ejecuta 

~~~
./install.sh
~~~

- Se te pedirá actualizar, elijes si

~~~
Use Zimbras package repository [Y] n
.
..
...
Do you wish to upgrade? [Y] y
.
..
...
Install zimbra-dnscache [N] n
.
..
...
Install zimbra-imapd (BETA - for evaluation only) [N] n
.
..
...
The system will be modified. Continue [N] y
~~~

- Al finalizar ejecutas yum/apt para asegurarte de que todos los paquetes y dependencias del sistema están correctamente actualizadas.

## Sugerencias
En Cuba utilizamos el mirror que tiene jovenclub, con el mismo puede dar problemas a la hora de actualizar, no así para instalar, al menos en los laboratorios que probé.

~~~
# Repositorio Joven club

#deb [arch=amd64] http://download.jovenclub.cu/repos/zimbra/apt/87 bionic zimbra
#deb [arch=amd64] http://download.jovenclub.cu/repos/zimbra/apt/90 bionic zimbra
~~~

Mi sugerencia es comentar el mismo y utilizar el oficial

~~~
deb https://repo.zimbra.com/apt/87 bionic zimbra
deb https://repo.zimbra.com/apt/90 bionic zimbra
~~~

## Errores luego de la actualización

HTTP ERROR 500 java.lang.NoClassDefFoundError: no se pudo inicializar la clase com.zimbra.soap.JaxbUtil

De ocurrirte este error, eso pasa por la versión de java que tienes instalada, en el la próxima entrada mostrare una solución