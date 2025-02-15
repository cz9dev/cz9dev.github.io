---
layout: post
title: Instalar MailAD en Debian
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
Al igual que en la configuración que realizamos en la variable enviroment anteriormente para que utilizara proxy, si estamos dentro de una empresa que utilice proxy para salir a internet deberiamos configurar wget para que salga a traves de un proxy. ¿Como ariamos esto?

1. Editamos el erchivo ```wgetrc``` que está dentro de /etc
```bash
nano /etc/wgetrc
# Dentro buscamos las configuraciones que pertenecen al proxy y las adaptamos a nuestra necesidad
# primero las descomentamos
https_proxy = http://proxy.mydomain.cu:3128/
http_proxy = http://proxy.mydomain.cu:3128/
ftp_proxy = http://proxy.mydomain.cu:3128/
```
2. Guardamos los cambios saliendo de nano con la convinacion de teclas Ctrl+X

### Configuramos a *apt* para que utilice proxy si fuera necesario
Así mismo si estamos dentro de una empresa que lo requiera configuramos *apt* para que utilice proxy, recuerde que si esas en un servidor de cara a internet estas configuraciones no son requeridas

1. Editamos o creamos el archivo *00proxy* que esta dentro de ```/etc/apt/apt.conf.d/00proxy``` de la siguiente forma
```bash
nano /etc/apt/apt.conf.d/00proxy
# Dentro agregamos estas configuraciones
Acqire::http::Proxy "http://proxy.mydomain.cu:3128";
Acqire::https::Proxy "http://proxy.mydomain.cu:3128";
Acqire::ftp::Proxy "http://proxy.mydomain.cu:3128";
```
2. Salimos de *nano* con Ctrl+X y guardamos los cambios
3. Hacemos Update y Upgrade de nuestro sistema y ya estaremos lismos para el proximo paso
```bash
apt update
.
..
...
apt upgrade
.
..
...

```
nota: si usted requiere credenciales en su proxy para conectarce a internet deberias en todas las configuraciones que hemos realizado cambiar *http://proxy.mydomain.cu:3128* por: *http://usuario:password@proxy.mydomain.cu*

### Instalamos paquetes necesarios para comenzar con la instalacion de MailAD
Para nuestra instalación se requiere de algunos paquetes en nustro sistemas, como son *git* para clonar el repositorio de MailAD (descargar mailad) y make para compilar.

1. Abrimos la terminal Ctrl+t.
```bash
apt update
apt upgrade
apt install git make -y
```

## Instalar MailAD
Ya estamos en condiciones de instalar MailAD.

1. Descargar MailAD e iniciar configuración

```bash
cd /root
git clone https://github.com/stdevPavelmc/mailad
cd mailad
git checkout master
git pull
```

2. Prepara tu servidor
```bash
make conf
```

Este paso creará la carpeta ```/etc/mailad``` y colocará un archivo ```mailad.conf ``` predeterminado en él. Ahora está listo, empieza a configurar su sistema.
Config inicial

Lea y llene todas las variables necesarias en ```/etc/mailad/mailad.conf``` por favor lea atentamente y elija sabiamente.

Si usted tiene Como servidor de Active directory Windows Server 2019 o superior, deberá modificar algunos parametros en los filtros que estan en la carpeta ```var/```del repositorio de MailAD, se explica en este [link](https://github.com/stdevPavelmc/mailad/blob/master/i18n/FAQ.es.md#he-instalado-seg%C3%BAn-las-instrucciones-todo-funciona-correctamente-pero-algunos-usuario-no-pueden-autenticarse-utilizo-windows-server-2019)

En este punto si quieres hacerlo todo rapido puede simplemente correr ```make all``` y seguir las pistas, con esta opcion *ya no tendras que hacer las que siguen* ya que todo quedará instalado y listo para el uso, pero si deceas hacerlo paso a paso sólo siguen los siguientes pasos

3. Manejar Dependencias

Instale las dependencias para instalar todas las herramientas necesarias

```bash
make deps
```

Esto instalará un grupo de herramientas necesarias para ejecutar los scripts de provisiones, si todo va bien no hay error que se muestre; si se muestra un error, entonces debe resolverlo ya que será el 99% del tiempo un problema relacionado con el enlace del repositorio y actualizaciones.

4. Comprobar configuración

Una vez que haya instalado las dependencias es el momento de comprobar la configuración local en busca de errores:

```bash
make conf-check
```

Esto comprobará si hay algunos de los escenarios y configuras predefinidos, si se encuentra algún problema se le advertirá al respecto.

Los errores mas comunes son:

Nombre de host: su servidor necesita saber su nombre de host completo calificado vea este tutorial para saber cómo resolver ese problema
* errores de ldapsearch: 100% del tiempo se debe a un error en el archivo mailad.conf, comprobarlo cuidadosamente

Estamos listos para instalar ahora... Oh, espera. Necesitamos generar los certificados SSL primero ;-)

5. Creación de certificados

Todas las comunicaciones del cliente deben estar cifradas, por lo que necesitará al menos un certificado autofirmado para uso interno. Este certificado será utilizado por postfix y dovecot.

Si usted procede el script MailAD generará un certificado autofirmado que durará 10 años, o si tiene certificados de Let's Encrypt (LE para abreviar) también puede utilizarlos, independientes y comodín son buenas opciones.

En caso de que tenga certificados LE, usarlos es simple. Sólo tienes que elegir los llamados "fullchain*" y "privkey*" y colóquelos en la carpeta /etc/mailad/le/, llántales fullchain.pemy privkey.pemrespectivamente, por lo que los guiones de provisiones podrían utilizarlos.

```bash
make certs
```

Los certificados finales se colocarán en este lugar (si está usando certificados LE serán copiados y asegurados):

* Certificado: ```/etc/ssl/certs/mail.crt```
* Clave privada: ```/etc/ssl/private/mail.key```
* Certificado de CA: ```/etc/ssl/certs/cacert.pem```

Si obtiene certificados LE para su servidor después del uso de los autofirmados, es necesario actualizarlos o reemplazarlos. Entonces sólo cargarlos (como nosotros describimos arriba) en el ```/etc/mailad/le/``` en la configuración y haga lo siguiente, en la carpeta que haya clonado la instalación de MailAD:

```bash
rm certs &2> /dev/null
make certs
systemctl restart postfix dovecot
systemctl status postfix dovecot
```

Los dos últimos pasos reinician los servicios relacionados con el correo electrónico y muestran su estado, para que puedas comprobar si todo salió bien. Si tienes problemas, simplemente elimina los archivos de la carpeta ```/etc/mailad/le/``` y repetir los pasos anteriores, que recrearán un certificado autofirmado y lo pondrían en servicio.

6. Instalaciones de software

```bash
make install
```

Este paso instala todo el software necesario.

7. Aprovicionamiento de servicios

Después de la instalación de software debe proporcionar la configuración, que se realiza con un solo comando:

```bash
make provision
```

Esta etapa copiará los archivos de plantilla en la carpeta de ```var``` de este repositorio sustituyendo los valores por los de su archivo ```mailad.conf```. Si se encuentra algún problema se le advertirá al respecto y tendrá que volver a ejecutar el comando ```make provision``` para continuar. También hay un ```make force-provision``` objetivo en caso de que necesite forzar la provisión a mano.

Cuando llegas a un mensaje de éxito después de la provisión estás listo para probar tu nuevo servidor de correo, **felicidades**.

8. Reconfiguración

Debe haber la necesidad algún tiempo en el futuro de cambiar algún parámetro de configuración de MailAD sin reinstalar o actualizar el servidor. El objetivo de hacer *force-provision* fue creado para eso, cambiar el parámetro(s) que desea en su archivo de configuración ```/etc/mailad/mailad.conf```, vaya a la carpeta de repo de MailAD ```cd /root/mailad``` (por defecto) y ejecute:

```bash
make force-provision
```

Usted verá como hace una copia de seguridad de toda la configuración y luego reinstale todo el servidor con los nuevos parámetros (este proceso durará unos 8 minutos en hardware actualizado). Está bien, ya que es la forma en que se desarrolla. Echa un vistazo a la última parte del proceso de instalación, verás algo como esto:

```bash
[...]
===> Latest backup is: /var/backups/mailad/20200912_033525.tar.gz
===> Extracting custom files from the backup...
etc/postfix/aliases/alias_virtuales
etc/postfix/rules/body_checks
etc/postfix/rules/header_checks
etc/postfix/rules/lista_negra
[...]
```

Sí, el ```force-provision``` así como el ```upgrade``` hacer que los objetivos conserven los datos modificados por el usuario.

Si necesita restablecer algunos de esos archivos a los valores predeterminados, simplemente borrarlos del sistema de archivos y hacer una disposición de fuerza, tan simple como eso.