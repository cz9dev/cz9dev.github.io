---
layout: post
title: Configurar wordpress detras de un servidor Proxy
subtitle:  Como configurar wordpress para que actualice o instale nuevos plugin detras de un proxy
##cover-img: /assets/img/posts/ssl.jpg
##thumbnail-img: /assets/img/posts/ssl.jpg
tags: [wordpress, proxy, php]
comments: true
---
En ocaciones tenemos nuestra instalación de wordpress detras de un servidor proxy, ejemplo de estos los sitios de intranet corporativas, entre otros. para esto es necesario decirle a nuestro wordpress que para instalar plugin o templay, debe pasar por un proxy, la forma para hacerlo es la siguiente:

1- Editamos el archivo wp-config.php que esta en la raíz de nuestro sitio.
``` sh
nano wp-config.php
```

2- Luego le agregamos al final del archivo lo siguiente:

```php
/** Configuración del proxy */
define('WP_PROXY_HOST', 'proxy.business.com');
define('WP_PROXY_PORT', '3128');
define('WP_PROXY_USERNAME', 'username');
define('WP_PROXY_PASSWORD', 'password');
define('WP_PROXY_BYPASS_HOSTS', 'localhost, 127.0.0.1, .business.com, 192.168.1.0/24');
```

En caso de que no necesites usuario y contraseña solo comenta estos, de la siguiente forma:
```php
/** Configuración del proxy */
define('WP_PROXY_HOST', 'proxy.business.com');
define('WP_PROXY_PORT', '3128');
//define('WP_PROXY_USERNAME', 'username');
//define('WP_PROXY_PASSWORD', 'password');
define('WP_PROXY_BYPASS_HOSTS', 'localhost, 127.0.0.1, .business.com, 192.168.1.0/24');
```

3- Guardas los cambios y actualiza tu pagina web.