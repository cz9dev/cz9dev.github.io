---
layout: post
title: Instalar zimbra 9 - zextras en cuba
subtitle: ¿Como instalar Zimbra9 - Zextras en Cuba utilizando Ubuntu 18.04?

thumbnail-img: /assets/img/zimbra_thumb.png
share-img: /assets/img/zimbra_path.png
tags: [ubuntu, zimbra, zextras, tutorial]
comments: true
---

Hola, les traigo en esta entrada como instalar el zimbra9 de zextras en ubuntu 18.04, puede ser tanto en una máquina virtual, un contenedor o en una instalación no virtualizada. Sin más pasamos al tutorial.

## Preparar el Sistema operativo para instalar zimbra

Primero que nada cambiamos de usuario para root
~~~
~# sudo su
root@zimbra-pc:/#
~~~

### Configuración de proxy (Si estubieras detras de uno)
Si estas detras de un proxy, configurar el sistema para que utilice proxy, de la siguiente forma:
~~~
cd /etc/apt/apt.conf.d
nano 00proxy
~~~

Ya dentro del archivo que estamos editando escribimos

~~~
Acquire::http::Proxy "http://ip_proxy:3128";
Acquire::https::Proxy "http://ip_proxy:3128";
Acquire::ftp::Proxy "http://ip_proxy:3128";
~~~

Salvamos los cambios y ahora editamos otro archivo para poder luego vajar nuestra key para el zimbra

~~~
cd /etc/
nano wgetrc
~~~

Localizamos en ese archivo donde dice https_proxy, http_proxy y ftp_proxy y lo hacemos conindir con nuestra configuracion de proxy, esas lineas estan al rededor de la linea 85

~~~
https_proxy = http://ip_proxy:3128/
http_proxy = http://ip_proxy:3128/
ftp_proxy = http://ip_proxy:3128/
~~~

Hasta aqui la parte de configuracion de proxy.

### Configuración de red

Si no hubieramos configurado la con el asistente de red o el dhcp nos hubiera entregado una ip, tendriamos que editar el siguiente fichero.

~~~
vi /etc/netplan/50-cloud-init.yaml
~~~

~~~
This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens192:
            addresses:
            - 192.168.1.205/24
            gateway4: 192.168.1.1
            nameservers:
                addresses:
                - [127.0.0.1,1.1.1.1]
    version: 2
~~~

Donde pone **addresses** habrá que cambiarlo por la IP que queramos poner y el gateway la dirección del router que tengamos en esa red, y de DNS Server deberemos poner o el propio servidor de Zimbra si vamos a usarlo de DNS, o un DNS externo.

Una ves que tengamos esto configurado, hacemos un apply de la configuración.

~~~
sudo netplan apply
~~~

Si quisieramos comprobar nuestra configuración (hacer un debug) hariamos lo siguiente:

~~~
sudo netplan --debug apply
** (generate:1962): DEBUG: 00:03:23.322: Processing input file /etc/netplan/50-cloud-init.yaml..
** (generate:1962): DEBUG: 00:03:23.322: starting new processing pass
** (generate:1962): DEBUG: 00:03:23.323: ens192: setting default backend to 1
** (generate:1962): DEBUG: 00:03:23.323: Configuration is valid
** (generate:1962): DEBUG: 00:03:23.323: Generating output files..
** (generate:1962): DEBUG: 00:03:23.323: NetworkManager: definition ens192 is not for us (backend 1)
DEBUG:netplan generated networkd configuration changed, restarting networkd
DEBUG:no netplan generated NM configuration exists
DEBUG:ens192 not found in {}
DEBUG:Merged config:
network:
  bonds: {}
  bridges: {}
  ethernets:
    ens192:
      addresses:
      - 192.168.1.205/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
        - [127.0.0.1,1.1.1.1]
  vlans: {}
  wifis: {}

DEBUG:Skipping non-physical interface: lo
DEBUG:device ens192 operstate is up, not changing
DEBUG:{}
DEBUG:netplan triggering .link rules for lo
DEBUG:netplan triggering .link rules for ens192
~~~

Debemos verificar que la configuracion de nuetro fuchero /etc/resolv.conf este bien configurado

~~~
nano /etc/resolv.conf
~~~

~~~
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 127.0.0.1
nameserver 8.8.8.8
~~~

Tambien deberiamos verificar que nuestro archivo hosts esté correctamente configurado:

~~~
nano /etc/hosts
~~~

Deberemos tener algo como lo siguiente, donde es importante que tengamos nuestra IP local, con el hostname+dominio y luego hostname

~~~
127.0.0.1       localhost
192.168.1.205   zcs.jorgedelacruz.es     zcs

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
~~~

Contodo esto estamos listos para reiniciar el equipo y continuar con el siguiente paso, para reiniciar el equipo hacemos lo siguiente:

~~~
reboot
~~~

### Instalación del servidor DNS

Instalaremos el servidor DNS dnsmasq, ideal para pequeños entornos. Es muy posible que tengamos instalado el systemd-resolved en nuestra pc Ubuntu 18.04, primero que nada lo desintalaremos para que no entre en conflicto con el dnsmasq

~~~
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
~~~

Podemos cambiar el nameserver 1.1.1.1 por otro servidor dns o incluso nuestro servidor DNS en la red

Para instalar el dnsmasq seguimos los pasos siguientes

~~~
apt-get install dnsmasq
~~~

Una ves instalado el dnsmasq editamos la configuracion para agregarle las entradas necesarias, al final del archivo agregamos:

~~~
nano /etc/dnsmasq.conf
~~~

~~~
server=1.1.1.1
listen-address=127.0.0.1
domain=midominio.cu
mx-host=midominio.cu,correo.midominio.cu,0
address=/correo.midominio.cu/192.168.1.5
~~~

Cerramos y guardamos los cambios en el fichero, posterior a esto reiniciamos el dnsmasq

~~~
service dnsmasq restart
* Restarting DNS forwarder and DHCP server dnsmasq
...done.
~~~

Antes de instalar zimbra es necesario comprobar que el MX resulva bien y que la entrada DNS de tipo A tambíen resulva, para esto utilizaremos la herramienta dig:

~~~
dig mx midominio.cu

; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> mx midominio.cu
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24828
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: 9aeacfcca1c268a1 (echoed)
;; QUESTION SECTION:
;midominio.cu.                        IN      MX

;; ANSWER SECTION:
midominio.cu.         3600    IN      MX      10 correo.midominio.cu.

;; ADDITIONAL SECTION:
correo.midominio.cu.  3600    IN      A       192.168.1.5

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Tue Jun 28 16:03:59 UTC 2022
;; MSG SIZE  rcvd: 64<code></code>
~~~

Ahora comprobamos el Host A:

~~~
dig correo.mudominio.cu

; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> correo.midominio.cu
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44493
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: 7fcf072a793ff523 (echoed)
;; QUESTION SECTION:
;correo.midominio.cu.         IN      A

;; ANSWER SECTION:
correo.midominio.cu.  3600    IN      A       192.168.1.5

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Tue Jun 28 16:07:38 UTC 2022
;; MSG SIZE  rcvd: 65<code></code>
~~~

con el test que realizamos podemos observar que la consulta devolvio un answer/respuesta y que el servidor ue respondio fue el local/127.0.0.1, si no optienes estos resultamos debes revizar la configuración de dnsmasq

## Instalacion de Zimbra 9 para Ubuntu 18.04 LTS – Zextras Build

Ya estamos en condiciones de instalar el zimbra, pero antes que nada desintalaremos el postfix que es muy probable que este instalado por defecto.

~~~
apt-get unistall postfix
~~~

Para nuestra instalación utilizaremos un mirror del repositorio de zimbra en Cuba, pero puede ser valido para el repositorio oficial u otro mirror. Vale aclarar que la instalacion desde un mirror será mucho mas lenta que desde el repo oficial.

### Configurar repositorio de zimbra
Cree el archivo /etc/apt/sources.list.d/zimbra.list con el contenido a continuación.

{% highlight sh linenos %}
deb [arch=amd64] http://download.jovenclub.cu/repos/zimbra/apt/87 bionic zimbra
deb [arch=amd64] http://download.jovenclub.cu/repos/zimbra/apt/90 bionic zimbra
{% endhighlight %}

Obtenga la llave GPG con: wget http://download.jovenclub.cu/repos/zimbra/zimbra-archive-keyring.gpg

Instale la llave con:

~~~
sudo apt-key add zimbra-archive-keyring.gpg
~~~

En muchos casos esta parte falla por que nuestro Ubuntu no tiene instalado el gnupg o gnupg2, para instalarlo seguimos estos pasos, y repetimos el paso anterior:

~~~
apt-get install gnupg2
~~~

Ejecute el siguiente comando para actualizar los índices de paquetes: sudo apt-get update

Una ves echo esto estamos en condicion de comenzamos con zimbra zextras build. Descargargamos el zimbra 9 - zextras desde en enlace oficial en /tmp/zcs y lo descomprimimo.

~~~
mkdir /tmp/zcs
cd /tmp/zcs
wget http://download.zextras.com/zcs-9.0.0_OSE_UBUNTU18_latest-zextras.tgz
~~~

Lo descomprimimos de la siguiente forma

~~~
tar xzvf zcs-9.0.0_OSE_UBUNTU18_latest-zextras.tgz
cd zcs-9.0.0_OSE_UBUNTU18_latest-zextras
~~~

Procedemos a instalar el Zimbra:

~~~
./install.sh

Operations logged to /tmp/install.log.hONgsH8e
Checking for existing installation...
    zimbra-drive...NOT FOUND
    zimbra-imapd...NOT FOUND
    zimbra-modern-ui...NOT FOUND
    zimbra-modern-zimlets...NOT FOUND
    zimbra-patch...NOT FOUND
    zimbra-mta-patch...NOT FOUND
    zimbra-proxy-patch...NOT FOUND
    zimbra-license-tools...NOT FOUND
    zimbra-license-extension...NOT FOUND
    zimbra-network-store...NOT FOUND
    zimbra-network-modules-ng...NOT FOUND
    zimbra-chat...NOT FOUND
    zimbra-connect...NOT FOUND
    zimbra-talk...NOT FOUND
    zimbra-ldap...NOT FOUND
    zimbra-logger...NOT FOUND
    zimbra-mta...NOT FOUND
    zimbra-dnscache...NOT FOUND
    zimbra-snmp...NOT FOUND
    zimbra-store...NOT FOUND
    zimbra-apache...NOT FOUND
    zimbra-spell...NOT FOUND
    zimbra-convertd...NOT FOUND
    zimbra-memcached...NOT FOUND
    zimbra-proxy...NOT FOUND
    zimbra-archiving...NOT FOUND
    zimbra-core...NOT FOUND
----------------------------------------------------------------------
PLEASE READ THIS AGREEMENT CAREFULLY BEFORE USING THE SOFTWARE.
SYNACOR, INC. ("SYNACOR") WILL ONLY LICENSE THIS SOFTWARE TO YOU IF YOU
FIRST ACCEPT THE TERMS OF THIS AGREEMENT. BY DOWNLOADING OR INSTALLING
THE SOFTWARE, OR USING THE PRODUCT, YOU ARE CONSENTING TO BE BOUND BY
THIS AGREEMENT. IF YOU DO NOT AGREE TO ALL OF THE TERMS OF THIS
AGREEMENT, THEN DO NOT DOWNLOAD, INSTALL OR USE THE PRODUCT.
License Terms for this Zimbra Collaboration Suite Software:
https://www.zimbra.com/license/zimbra-public-eula-2-6.html
----------------------------------------------------------------------
Do you agree with the terms of the software license agreement? [N] y
~~~

En este punto seleccionaremos "y" si aceptamos la EULA que nos indica el enlace, de no aceptar la EULA no podremos seguir nuestra instalación.

~~~
Use Zimbra's package repository [Y]
~~~

Esta parte es muy importante, instalaremos solo los paquetes que nos interece, ademas lo aremos desde el repo mirror que configuramos. si escojemos "y" utilizaremos los paquetes del repositorio oficial por lo que escojeremos "n" para utilizar los paquetes del repo mirror que configuramos.

~~~
Use Zimbra's package repository [Y] n

Configuring package repository

Checking for installable packages

Found zimbra-core (local)
Found zimbra-ldap (local)
Found zimbra-logger (local)
Found zimbra-mta (local)
Found zimbra-dnscache (local)
Found zimbra-snmp (local)
Found zimbra-store (local)
Found zimbra-apache (local)
Found zimbra-spell (local)
Found zimbra-memcached (repo)
Found zimbra-proxy (local)
Found zimbra-drive (repo)
Found zimbra-imapd (local)
Found zimbra-patch (repo)
Found zimbra-mta-patch (repo)
Found zimbra-proxy-patch (repo)


Select the packages to install
Install zimbra-ldap [Y] 
Install zimbra-logger [Y] 
Install zimbra-mta [Y] 
Install zimbra-dnscache [Y] n
Install zimbra-snmp [Y] 
Install zimbra-store [Y] 
Install zimbra-apache [Y] 
Install zimbra-spell [Y] 
Install zimbra-memcached [Y] 
Install zimbra-proxy [Y] 
Install zimbra-drive [Y] 
Install zimbra-imapd (BETA - for evaluation only) [N] 
Install zimbra-chat [Y] 
Checking required space for zimbra-core
Checking space for zimbra-store
Checking required packages for zimbra-store
zimbra-store package check complete.

Installing:
    zimbra-core
    zimbra-ldap
    zimbra-logger
    zimbra-mta
    zimbra-snmp
    zimbra-store
    zimbra-apache
    zimbra-spell
    zimbra-memcached
    zimbra-proxy
    zimbra-drive
    zimbra-patch
    zimbra-mta-patch
    zimbra-proxy-patch
    zimbra-chat

The system will be modified.  Continue? [N] y
~~~

Precionamos "y" para modificar el sistema:

~~~
Beginning Installation - see /tmp/install.log.hONgsH8e for details...

                          zimbra-core-components will be downloaded and installed.
                            zimbra-timezone-data will be installed.
                          zimbra-common-core-jar will be installed.
                         zimbra-common-mbox-conf will be installed.
                    zimbra-common-mbox-conf-msgs will be installed.
                   zimbra-common-mbox-conf-attrs will be installed.
                  zimbra-common-mbox-conf-rights will be installed.
                         zimbra-common-mbox-docs will be installed.
                           zimbra-common-mbox-db will be installed.
                         zimbra-common-core-libs will be installed.
                   zimbra-common-mbox-native-lib will be installed.
                                     zimbra-core will be installed.
                          zimbra-ldap-components will be downloaded and installed.
                                     zimbra-ldap will be installed.
                                   zimbra-logger will be installed.
                           zimbra-mta-components will be downloaded and installed.
                                      zimbra-mta will be installed.
                          zimbra-snmp-components will be downloaded and installed.
                                     zimbra-snmp will be installed.
                         zimbra-store-components will be downloaded and installed.
                       zimbra-jetty-distribution will be downloaded and installed.
                                 zimbra-mbox-war will be installed.
                       zimbra-mbox-webclient-war will be installed.
                                zimbra-mbox-conf will be installed.
                             zimbra-mbox-service will be installed.
                   zimbra-mbox-admin-console-war will be installed.
                          zimbra-mbox-store-libs will be installed.
                                    zimbra-store will be installed.
                        zimbra-apache-components will be downloaded and installed.
                                   zimbra-apache will be installed.
                         zimbra-spell-components will be downloaded and installed.
                                    zimbra-spell will be installed.
                                zimbra-memcached will be downloaded and installed.
                         zimbra-proxy-components will be downloaded and installed.
                                    zimbra-proxy will be installed.
                                    zimbra-drive will be downloaded and installed (later).
                                    zimbra-patch will be downloaded and installed (later).
                                zimbra-mta-patch will be downloaded and installed (later).
                              zimbra-proxy-patch will be downloaded and installed (later).
                                     zimbra-chat will be downloaded and installed (later).

Downloading packages (10):
   zimbra-core-components
   zimbra-ldap-components
   zimbra-mta-components
   zimbra-snmp-components
   zimbra-store-components
   zimbra-jetty-distribution
   zimbra-apache-components
   zimbra-spell-components
   zimbra-memcached
   zimbra-proxy-components
      ...done

Removing /opt/zimbra
Removing zimbra crontab entry...done.
Cleaning up zimbra init scripts...done.
Cleaning up /etc/security/limits.conf...done.

Finished removing Zimbra Collaboration Server.


Installing repo packages (10):
   zimbra-core-components
   zimbra-ldap-components
   zimbra-mta-components
   zimbra-snmp-components
   zimbra-store-components
   zimbra-jetty-distribution
   zimbra-apache-components
   zimbra-spell-components
   zimbra-memcached
   zimbra-proxy-components
      ...done

Installing local packages (25):
   zimbra-timezone-data
   zimbra-common-core-jar
   zimbra-common-mbox-conf
   zimbra-common-mbox-conf-msgs
   zimbra-common-mbox-conf-attrs
   zimbra-common-mbox-conf-rights
   zimbra-common-mbox-docs
   zimbra-common-mbox-db
   zimbra-common-core-libs
   zimbra-common-mbox-native-lib
   zimbra-core
   zimbra-ldap
   zimbra-logger
   zimbra-mta
   zimbra-snmp
   zimbra-mbox-war
   zimbra-mbox-webclient-war
   zimbra-mbox-conf
   zimbra-mbox-service
   zimbra-mbox-admin-console-war
   zimbra-mbox-store-libs
   zimbra-store
   zimbra-apache
   zimbra-spell
   zimbra-proxy
      ...done

Installing extra packages (5):
   zimbra-drive
   zimbra-patch
   zimbra-mta-patch
   zimbra-proxy-patch
   zimbra-chat
      ...done

Running Post Installation Configuration:
Operations logged to /tmp/zmsetup.20200718-195904.log
Installing LDAP configuration database...done.
Setting defaults...
~~~

Ahora tendremos que cambiar el dominio por defecto, es donde mas se suele fallar:

~~~
DNS ERROR resolving MX for correo.midominio.cu
It is suggested that the domain name have an MX record configured in DNS
Change domain name? [Yes] yes
Create domain: [correo.midominio.cu] midominio.cu
        MX: correo.midominio.cu (192.168.1.4)

        Interface: 127.0.0.1
        Interface: ::1
        Interface: 192.168.1.5
done.
~~~

Cambiamos la contraseña de admin. Para hacerlo entramos al menú 6 del principal y luego al submenu 4:

~~~
Select, or 'r' for previous menu [r] 4
~~~

Escribimos la contraseña que queramos:

~~~
Password for admin@midominio.cu (min 6 characters): [LuUUyHNhE] Zimbra2022

Store configuration

   1) Status:                                  Enabled                       
   2) Create Admin User:                       yes                           
   3) Admin user to create:                    admin@jorgedelacruz.es        
   4) Admin Password                           set                           
   5) Anti-virus quarantine user:              virus-quarantine.os8g3ngt@jorgedelacruz.es
   6) Enable automated spam training:          yes                           
   7) Spam training user:                      spam.r_bwdt3yj@jorgedelacruz.es
   8) Non-spam(Ham) training user:             ham.kntjdvuqcq@jorgedelacruz.es
   9) SMTP host:                               zcs9zx.jorgedelacruz.es       
  10) Web server HTTP port:                    8080                          
  11) Web server HTTPS port:                   8443                          
  12) Web server mode:                         https                         
  13) IMAP server port:                        7143                          
  14) IMAP server SSL port:                    7993                          
  15) POP server port:                         7110                          
  16) POP server SSL port:                     7995                          
  17) Use spell check server:                  yes                           
  18) Spell server URL:                        http://zcs9zx.jorgedelacruz.es:7780/aspell.php
  19) Enable version update checks:            TRUE                          
  20) Enable version update notifications:     TRUE                          
  21) Version update notification email:       admin@jorgedelacruz.es        
  22) Version update source email:             admin@jorgedelacruz.es        
  23) Install mailstore (service webapp):      yes                           
  24) Install UI (zimbra,zimbraAdmin webapps): yes                           

Select, or 'r' for previous menu [r]
~~~

Pulsamos ** enter ** para regrezar al menú principal:

~~~
Select, or 'r' for previous menu [r] 

Main menu

   1) Common Configuration:                                                  
   2) zimbra-ldap:                             Enabled                       
   3) zimbra-logger:                           Enabled                       
   4) zimbra-mta:                              Enabled                       
   5) zimbra-snmp:                             Enabled                       
   6) zimbra-store:                            Enabled                       
   7) zimbra-spell:                            Enabled                       
   8) zimbra-proxy:                            Enabled                       
   9) Default Class of Service Configuration:                                
   s) Save config to file                                                    
   x) Expand menu                                                            
   q) Quit                                    

*** CONFIGURATION COMPLETE - press 'a' to apply
Select from menu, or press 'a' to apply config (? - help) a
~~~

Si precionamos a, aplicamos los cambios:

~~~
Select from menu, or press 'a' to apply config (? - help) a
~~~

Precionamos * enter *

~~~
Save configuration data to a file? [Yes]
~~~

Precionamos * enter *

~~~
Save config in file: [/opt/zimbra/config.10687]
Saving config in /opt/zimbra/config.10687...done.
~~~

Precionamos "y" para continuar

~~~
Operations logged to /tmp/zmsetup.20200718-195904.log
Setting local config values...done.
Initializing core config...Setting up CA...done.
Deploying CA to /opt/zimbra/conf/ca ...done.
Creating SSL zimbra-store certificate...done.
Creating new zimbra-ldap SSL certificate...done.
Creating new zimbra-mta SSL certificate...done.
Creating new zimbra-proxy SSL certificate...done.
Installing mailboxd SSL certificates...done.
Installing MTA SSL certificates...done.
Installing LDAP SSL certificate...done.
Installing Proxy SSL certificate...done.
Initializing ldap...done.
Setting replication password...done.
Setting Postfix password...done.
Setting amavis password...done.
Setting nginx password...done.
Setting BES searcher password...done.
Creating server entry for zcs9zx.jorgedelacruz.es...done.
Setting Zimbra IP Mode...done.
Saving CA in ldap...done.
Saving SSL Certificate in ldap...done.
Setting spell check URL...done.
Setting service ports on zcs9zx.jorgedelacruz.es...done.
Setting zimbraFeatureTasksEnabled=TRUE...done.
Setting zimbraFeatureBriefcasesEnabled=TRUE...done.
Checking current setting of zimbraReverseProxyAvailableLookupTargets
Querying LDAP for other mailstores
Searching LDAP for reverseProxyLookupTargets...done.
Adding zcs9zx.jorgedelacruz.es to zimbraReverseProxyAvailableLookupTargets
Updating zimbraLDAPSchemaVersion to version '1583314207'
Setting TimeZone Preference...done.
Disabling strict server name enforcement on zcs9zx.jorgedelacruz.es...done.
Initializing mta config...done.
Setting services on zcs9zx.jorgedelacruz.es...done.
Adding zcs9zx.jorgedelacruz.es to zimbraMailHostPool in default COS...done.
Creating domain jorgedelacruz.es...done.
Setting default domain name...done.
Creating domain jorgedelacruz.es...already exists.
Creating admin account admin@jorgedelacruz.es...done.
Creating root alias...done.
Creating postmaster alias...done.
Creating user spam.r_bwdt3yj@jorgedelacruz.es...done.
Creating user ham.kntjdvuqcq@jorgedelacruz.es...done.
Creating user virus-quarantine.os8g3ngt@jorgedelacruz.es...done.
Setting spam training and Anti-virus quarantine accounts...done.
Initializing store sql database...done.
Setting zimbraSmtpHostname for zcs9zx.jorgedelacruz.es...done.
Configuring SNMP...done.
Setting up syslog.conf...done.
Starting servers...done.
Installing common zimlets...
    com_zextras_chat_open...done.
    com_zimbra_ymemoticons...done.
    com_zimbra_bulkprovision...done.
    com_zimbra_clientuploader...done.
    com_zimbra_viewmail...done.
    com_zimbra_attachcontacts...done.
    com_zimbra_email...done.
    com_zimbra_url...done.
    com_zimbra_srchhighlighter...done.
    com_zextras_drive_open...done.
    com_zimbra_tooltip...done.
    com_zimbra_cert_manager...done.
    com_zimbra_phone...done.
    com_zimbra_attachmail...done.
    com_zimbra_proxy_config...done.
    com_zimbra_webex...done.
    com_zimbra_mailarchive...done.
    com_zimbra_adminversioncheck...done.
    com_zimbra_date...done.
Finished installing common zimlets.
Restarting mailboxd...done.
Creating galsync account for default domain...done.

You have the option of notifying Zimbra of your installation.
This helps us to track the uptake of the Zimbra Collaboration Server.
The only information that will be transmitted is:
    The VERSION of zcs installed (9.0.0_ZEXTRAS_202007114_UBUNTU18_64)
    The ADMIN EMAIL ADDRESS created (admin@jorgedelacruz.es)

Notify Zimbra of your installation? [Yes] 
Notifying Zimbra of installation via http://www.zimbra.com/cgi-bin/notify.cgi?VER=9.0.0_ZEXTRAS_202007114_UBUNTU18_64&MAIL=admin@midominio.cu

Notification complete
Checking if the NG started running...done. 
Setting up zimbra crontab...done.
Moving /tmp/zmsetup.20200718-195904.log to /opt/zimbra/log
Configuration complete - press return to exit
~~~

Ya culminamos nuestra instalación. Ya podemos agregar nustros usuarios y empezar a utilizar nuestro servidor de correo, podemos agrear usuarios de un LDAP o AD existente y sincronizar con los mismos, pero eso sera visto en otra entrada del blog.

Esta entrada de blog tiene como fuente rincipal el blog de [jorge de la cruz](jorgedelacruz.es) y el de [sisadmmindecuba](https://www.sysadminsdecuba.com/), así que les recomiendo leer el articulo que les comparto con el enlace siguiente

## ¿Qué hacer una vez instalado?

https://www.jorgedelacruz.es/2015/01/13/zimbra-una-vez-instalado-que-hago-administrador/