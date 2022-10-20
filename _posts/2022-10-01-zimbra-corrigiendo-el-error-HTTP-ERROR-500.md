---
layout: post
title: Como corregir el error HTTP ERROR 500 java.lang.NoClassDefFoundError
subtitle: Solucion para el error HTTP ERROR 500 luego de actualizar Zimbra
cover-img: /assets/img/zimbra_path.png
thumbnail-img: /assets/img/zimbra_thumb.png
tags: [ubuntu, zimbra, zextras, tutorial]
comments: true
---

En esta entrada, me gustaría compartir como corregir del error ***"HTTP ERROR 500 java.lang.NoClassDefFoundError: no se pudo inicializar la clase com.zimbra.soap.JaxbUtil"*** que se puede mostrar al acceder a Zimbra después de actualizar a Zimbra 9 bluid Zextras Release 9.0.0.ZEXTRAS.20220713.UBUNTU18.64 UBUNTU18_64 FOSS edition o después de aplicar un parche a la versión 8.8.15 (Network Edition y Open Source).

La solución para este error, que se debe a que el script de actualización no puede actualizar el parámetro "mailboxd_java_options" de zmlocalconfig, es agregar las siguientes opciones:

> -Djava.security.egd=file:/dev/./urandom --add-opens java.base/java.lang=TODO-SIN NOMBRE

En este ejemplo, vemos cómo se ve “mailboxd_java_options” sin la corrección del parámetro:

> mailboxd_java_options = -server -Dhttps.protocols=TLSv1.2,TLSv1.3 -Djdk.tls.client.protocols=TLSv1.2,TLSv1.3 -Djava.awt.headless=true -Dsun.net.inetaddr.ttl=${networkaddress_cache_ttl} -Dorg.apache.jasper.compiler.disablejsr199=true -XX:+UseG1GC -XX:SoftRefLRUPolicyMSPerMB=1 -XX:+UnlockExperimentalVMOptions -XX:G1NewSizePercent=15 -XX:G1MaxNewSizePercent=45 -XX:-OmitStackTraceInFastThrow -verbose:gc -Xlog:gc*=info,safepoint=info:file=/opt/zimbra/log/gc.log:time:filecount=20,filesize=10m -Djava.net.preferIPv4Stack=true

Ahora utilizando el comando zmlocalconfig -e para agregar la opcion

~~~
zmlocalconfig -e mailboxd_java_options="-server -Dhttps.protocols=TLSv1.2,TLSv1.3 -Djdk.tls.client.protocols=TLSv1.2,TLSv1.3 -Djava.awt.headless=true -Dsun.net.inetaddr.ttl=${networkaddress_cache_ttl} -Dorg.apache.jasper.compiler.disablejsr199=true -XX:+UseG1GC -XX:SoftRefLRUPolicyMSPerMB=1 -XX:+UnlockExperimentalVMOptions -XX:G1NewSizePercent=15 -XX:G1MaxNewSizePercent=45 -XX:-OmitStackTraceInFastThrow -verbose:gc -Xlog:gc*=info,safepoint=info:file=/opt/zimbra/log/gc.log:time:filecount=20,filesize=10m -Djava.net.preferIPv4Stack=true -Djava.security.egd=file:/dev/./urandom --add-opens java.base/java.lang=ALL-UNNAMED"
zmmailboxdctl restart
~~~

En el ejemplo siguiente , vemos cómo se ve “mailboxd_java_options” con la corrección del parámetro:

> mailboxd_java_options = -server -Dhttps.protocols=TLSv1.2,TLSv1.3 -Djdk.tls.client.protocols=TLSv1.2,TLSv1.3 -Djava.awt.headless=true -Dsun.net.inetaddr.ttl=${networkaddress_cache_ttl} -Dorg.apache.jasper.compiler.disablejsr199=true -XX:+UseG1GC -XX:SoftRefLRUPolicyMSPerMB=1 -XX:+UnlockExperimentalVMOptions -XX:G1NewSizePercent=15 -XX:G1MaxNewSizePercent=45 -XX:-OmitStackTraceInFastThrow -verbose:gc -Xlog:gc*=info,safepoint=info:file=/opt/zimbra/log/gc.log:time:filecount=20,filesize=10m -Djava.net.preferIPv4Stack=true -Djava.security.egd=file:/dev/./urandom --add-opens java.base/java.lang=ALL-UNNAMED