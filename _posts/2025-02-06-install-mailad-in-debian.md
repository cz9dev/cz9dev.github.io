---
layout: post
title: Inslatar MailAD en Debian
subtitle: ¿Como instalar MailAD en debian y no morir en el intento?
tags: [MailAD, Proxmox, Debian]
comments: true
---

En esta entrada les traigo un tutorial de como instalar MailAD en Debian, pero primero vamos a dar una breve introducción al asunto. Para los que no conocen que es MailAD. MailAD es una receta, formula, forma, compilación o integración de programas para implementar e implantar un servidor de correo, desarrollado por "Pavel Milanes" @pavelmc, desarrollador cubano, detrás existe una comunidad de desarrolladores que aportan a su mantenimiento y estabilidad. Esta desarrollado para un rápido despliegue e integración con un Servidor Active Directory. Los software fundamentales que despliega MailAd para su funcionamiento son [postfix](https://www.postfix.org/) y [dovecot](https://www.dovecot.org/), también despliega un servicio para webmail que puede ser [roundcube](https://https://roundcube.net) por defecto o [snappymail](https://snappymail.eu). Sin mas preámbulos comenzamos.

# Instalación y configuración de pre-requisitos para la instalación de MailAD
Para la instalación de MailAD estaremos utilizado el repositorio en github.

## Preparación de nuestro Servidor Active Directory

### Crear usuario para mapeara usuarios de AD
Primero que nada crearemos un usuario para el mapear los usuario de nuestro servidor Active Directory, para nuestro ejemplo le llamaremos el usuario será: ```mail```, le pondremos como contraseña: ```Passw0rd---```, el usuario lo crearemos en la unidad organizativa Users no en nuestra unidad organizativa. Será miembro del grupo de seguridad **Administrador de esquema**, a continuación una foto con el usuario y otra con el grupo a que pertenecerá para ayudar a una mejor compresión:
![Usuario mapear usuarios de AD](/assets/img/install-mailad/usuario-para-mapear-usuarios-ad.png "Usuario mapear usuarios de AD")

![Grupos a los que pertenece el usuario](/assets/img/install-mailad/grupos-usuario-mapear-ad.png "Grupos a los que pertenece el usuario")

### Crear estructura de unidades organizativas y grupos de seguridad
Crearemos dentro de nuestra unidad organizativa una unidad organizativa con el nombre exacto **MAIL_ACCESS**, este nombre no puede variar, de lo contrario no funcionará correctamente, dentro de **MAIL_ACCESS** crearemos dos grupos de seguridad uno llamado *Local_mail* y otro *National_mail*, estos dos grupos de seguridad como su nombre lo indica son para según corresponda a los usuarios con correo local asignarles el grupo *Local_mail* y el *National_mail* para los usuarios con correo nacional, los usuarios que no estén en ninguno de los dos grupos tendrán correo internacional
![Estructura organizativa](/assets/img/install-mailad/estructura-organizativa-mailad.png "Unidades organizativas y Grupos de Seguridad")

### Crear o modificar propiedades en el o los usuarios
A los usuarios en las propiedades generales se le debe agregar el correo electrónico de la empresa que será el reconocido por **MailAD**
![Configuración del campo correo en los usuarios](/assets/img/install-mailad/configuracion-de-campo-correo-en-usuario.png "Configuración del campo de correo en los usuarios")

nota: si creas dentro de tu unidad organizativa principal grupos de seguridad y a estos le agregas la propiedad de correo electrónico te funciona como listas de correo, un ejemplo seria, un grupo de seguridad *GrupoEconomico* y a este agregarle la propiedad de correo *grupo.economico@mydomain.cu*, todos los usuarios dentro de este grupo estarían en la lista de correo *grupo.economico@myadomain.cu*.

## Preparación de nuestro DNS
1. Crearemos en nuestro *DNS* una host(A o AAAA) apuntando al ip de nuestro servidor de correo
2. Creamos los alias (CNAME) que creamos conveniente en para nuestro correo, podrían ser: pop, smtp, imap, webmail, apuntando a nuestro hosts de correo
3. Creamos un registro de intercambio de correo (MX) apuntando a nuestro hosts de correo

## Preparación de nuestro Sistema Operativo Debian antes de comenzar con la instalación de MailAD

### Crear o modificar variable de entorno para utilizar salida de internet de nuestro servidor de correo a traves de un PROXY
Este no es un punto que sea obligatorio, pero si estuviéramos instalando nuestro servidor de correo en una institución o empresa que salga a internet mediante un servidor proxy, sería necesario este paso

Modificar el archivo enviroment de linux y agregar la configuración de proxy como sigue en el ejemplo

```bash
nano /etc/environment
## Agregar el final del archivo estas propiedades
HTTP_PROXY=http://192.168.1.1:3128
HTTPS_PROXY=http://192.168.1.1:3128
FTP_PROXY=http://192.168.1.1:3128
NO_PROXY=localhost,127.0.0.1,192.168.1.0/24,.mydomain.cu
http_proxy=http://192.168.1.1:3128
https_proxy=http://192.168.1.1:3128
ftp_proxy=http://192.168.1.1:3128
no_proxy=localhost,127.0.0.1,192.168.1.0/24,.mydomain.cu
```
Recuerde cambiar *192.168.1.1* por el ip del servidor proxy de su red, y *mydomain.cu* por el dominio de su red

### Modificar la configuración proxy de wget
Al igual que el 