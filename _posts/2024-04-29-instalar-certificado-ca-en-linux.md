---
layout: post
title: Instalar certificado raíz en Linux (Debian, Ububtu, Linux Mint)
subtitle:  ¿Cómo podemos instalar un certificado raíz en distribuciones linux?
##cover-img: /assets/img/posts/ssl.jpg
thumbnail-img: /assets/img/posts/ssl.jpg
tags: [Linux, Debian, Debuan, Ubuntu, Linux Mint]
comments: true
---


Dado un archivo de certificado de CA foo.crt, Seguir los siguientes pasos para instalarlos:

Crear un directorio para certificados de CA adicionales en /usr/local/share/ca-certificates, aunque no es necesario, pueden ponerce los certificados en el directorio ca-certificates:

``` sh
sudo mkdir /usr/local/share/ca-certificates/extra
```

Copiar el certificado CA .crt file a este directorio:

``` sh
sudo cp foo.crt /usr/local/share/ca-certificates/extra/foo.crt
```

Deja que Ubuntu agregue el .crt la ruta del archivo en relación con /usr/local/share/ca-certificates a /etc/ca-certificates.conf:

``` sh
sudo dpkg-reconfigure ca-certificates
``` 

Para hacer esto de manera no interactiva, corre:

``` sh
sudo update-ca-certificates
```

En el caso de una .pem primero debe ser convertido a un .crt:

``` sh
openssl x509 -in foo.pem -inform PEM -out foo.crt
```

O si es un archivo .cer convertirlo a .crt:

``` sh
openssl x509 -inform DER -in foo.cer -out foo.crt
```