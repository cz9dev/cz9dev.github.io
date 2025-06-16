---
layout: post
title: Guía Completa para Desplegar tu Aplicación Node.js en Producción (Auto-hospedado)
subtitle: Como desplegar tu Aplicación Node.js en Producción (Auto-hospedado)
##cover-img: /assets/img/posts/self-hoste-nodejs.png
thumbnail-img: /assets/img/posts/self-hoste-nodejs.png
tags: [Node.js]
comments: true
---

Para poner tu aplicación Node.js en producción de forma auto-hospedada, sigue estos pasos detallados:

# Preparación del Entorno de Producción

## Requisitos básicos:
* Servidor físico/VPS: Puedes usar un servidor dedicado, VPS (como DigitalOcean, Linode, OVH) o incluso una máquina local
* Sistema Operativo: Ubuntu Server LTS (recomendado) o CentOS
* Conexión SSH: Para administración remota

## Configuración Inicial del Servidor

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar herramientas básicas
sudo apt install -y git curl wget build-essential

# Instalar Node.js (usando nvm para mejor gestión de versiones)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
```

## Instalar y Configurar Base de Datos

Para MongoDB (si es tu base de datos):

```bash
sudo apt install -y mongodb
sudo systemctl enable mongodb
sudo systemctl start mongodb
```

## Crear usuario admin (ejecutar en mongo shell)

```bash
mongo
> use admin
> db.createUser({user: "admin", pwd: "passwordSeguro", roles: ["root"]})
> exit
```

## Desplegar tu Aplicación

Opción A: Clonar repositorio Git

```bash
# Clonar tu repositorio
git clone https://github.com/tu-usuario/tu-repo.git
cd tu-repo

# Instalar dependencias
npm install --production

# Configurar variables de entorno
cp .env.example .env
nano .env  # Editar con tus valores reales
```

Opción B: Subir archivos manualmente

Usa SFTP/SCP (como FileZilla) para subir tus archivos al servidor.

## Configurar Reverse Proxy con Nginx

```bash

# Instalar Nginx
sudo apt install -y nginx

# Configurar sitio
sudo nano /etc/nginx/sites-available/tu-app
```

Configuración básica de Nginx:

```nginx
server {
    listen 80;
    server_name tudominio.com www.tudominio.com;

    location / {
        proxy_pass http://localhost:3000;  # Puerto donde corre tu app
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Opcional: Configuración para archivos estáticos
    location /public/ {
        root /ruta/a/tu/app;
        expires 30d;
    }
}

```

Activar la configuración:

```bash

sudo ln -s /etc/nginx/sites-available/tu-app /etc/nginx/sites-enabled
sudo nginx -t  # Verificar sintaxis
sudo systemctl restart nginx
```

## Gestionar Procesos con PM2

```bash

# Instalar PM2 globalmente
npm install -g pm2

# Iniciar aplicación
pm2 start server.js --name "tu-app"  # Reemplaza server.js por tu archivo principal

# Configurar inicio automático
pm2 startup
pm2 save
```

## Configurar Firewall (UFW)

```bash

sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```

## Configurar Certificado SSL (HTTPS)

Usemos Certbot para certificados Let's Encrypt gratuitos:

```bash

sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d tudominio.com -d www.tudominio.com
```

## Monitoreo y Mantenimiento

Monitoreo básico:

```bash

pm2 monit  # Monitor en tiempo real
pm2 logs   # Ver logs
```

### Actualizaciones automáticas (opcional):

Crear un script /usr/local/bin/update-app:

```bash

#!/bin/bash
cd /ruta/a/tu/app
git pull origin main
npm install --production
pm2 restart tu-app
```

Hacerlo ejecutable:
bash

```bash
sudo chmod +x /usr/local/bin/update-app
```

## Copias de Seguridad
Backup básico de base de datos (cron job):
bash

### Crear script de backup

```bash
sudo nano /usr/local/bin/backup-db
```

Contenido del script:

```bash

#!/bin/bash
DATE=$(date +%Y-%m-%d)
mongodump --uri="mongodb://admin:passwordSeguro@localhost:27017" --out=/backups/mongodb-$DATE
find /backups/ -type f -mtime +7 -delete
```

Programar en cron (crontab -e):

```text

0 3 * * * /usr/local/bin/backup-db
```

## Opciones Avanzadas
### Dockerizar tu aplicación (recomendado para aislamiento):

1. Crear Dockerfile

2. Construir imagen: docker build -t tu-app .

3. Ejecutar contenedor: docker run -d -p 3000:3000 --name tu-app tu-app

### Balanceo de carga:

Si necesitas escalar horizontalmente, considera:
* Usar múltiples instancias con PM2 cluster mode
* Configurar Nginx como balanceador de carga

## Solución de Problemas Comunes

1. La aplicación no se inicia:

* Verifica logs: pm2 logs tu-app --lines 100

* Revisa puertos: sudo netstat -tulnp | grep node

2. Problemas de conexión a la base de datos:

* Verifica que MongoDB esté corriendo: sudo systemctl status mongodb

* Comprueba credenciales en .env

3. Errores 502 Bad Gateway:

* Verifica que tu app esté corriendo: pm2 list

* Revisa logs de Nginx: sudo tail -f /var/log/nginx/error.log

Con esta configuración tendrás una aplicación Node.js en producción robusta, segura y con capacidad de crecimiento.