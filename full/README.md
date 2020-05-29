# Instrucciones

## Levantar stack
   - Crear imagenes
   - Arrancar servicios
   - Añadir entradas a /etc/hosts (opcional)

```shell
docker-compose build

docker-compose up -d

echo "127.0.0.1 drupal.docker.localhost" | sudo tee -a /etc/hosts
```

## Comprobar los servicios levantados

```shell
docker-compose ps
```

## Comprobar logs
```shell
docker-compose logs

docker-compose logs -f nginx
```
## Información de un servicio

```shell
docker inspect my_drupal8_project_nginx

docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' my_drupal8_project_nginx

docker inspect my_drupal8_project_nginx | jq .[0] | jq keys

docker inspect my_drupal8_project_nginx | jq .[0].State

docker inspect my_drupal8_project_nginx | jq -r '.[].NetworkSettings.Networks[].IPAddress'
```

## Bootstrap drupal
   - creación de projecto drupal con composer

```shell
docker exec my_drupal8_project_php composer create-project drupal-composer/drupal-project:8.x-dev /var/www/html --no-interaction
```

- Acceder a la url 'drupal.docker.localhost' y realizar instalación
    - Comprobar valores para la base de datos en el fichero '.env'

## Comprobar los ficheros de configuración de los servicios destacados en este stack:
    - Drupal: /var/www/html/web/sites/default/settings.php, /var/www/html/web/sites/sites.php
    - Haproxy: /usr/local/etc/haproxy/haproxy.cfg
    - Varnish: /etc/varnish/includes/backend.vcl, /etc/varnish/default.vcl
    - Nginx: /etc/nginx/drupal.conf, /etc/nginx/conf.d/default.conf

## Backups

```shell
source .env

MYSQL_HOST=$(docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' ${PROJECT_NAME}_mariadb)

mysqldump --single-transaction --quick -Be --host=${MYSQL_HOST} --user=${DB_USER} --password=${DB_PASSWORD} ${DB_NAME} > ${DB_NAME}_dump_$(date +'%Y%m%d%H%M').sql
```

## Actualizacion del drupal core

0. Acceder al directorio donde tengamos los ficheros de composer (/var/www/html)

```shell
docker exec -ti my_drupal8_project_php bash
```

1. Comprobar que existen paquetes sin actualizar

```shell
composer outdated
```

2. Mostrar paquetes sin actualizar del core de drupal

```shell
composer outdated drupal/*
```

3. Colocar drupal en modo mantenimiento y borrar la cache

```shell
drush sset system.maintenance_mode 1
drush cr
```

4. Realizar la actualización del código y las dependencias

```shell
composer update Drupal/core –with-dependencies
```

5. Actualizar el esquema de la base de datos (no siempre será necesario)

```shell
drush updb
```

Podemos comprobar el proceso de actualización y posibles errores en los siguientes apartados de la web

reports->available

reports->status

6. Sacar el portal del modo mantenimiento y volver a borrar la cache

```shell
drush sset system.maintenance_mode 0
drush cr
```

7. Comprobar estado del site

```shell
drush status
```

## Añadir configuración de drupal para memcached

El fichero settings.local.php es cargado por defecto si se encuentra en el mismo directorio que settings.php

```shell
docker cp settings.local.php my_drupal8_project_php:/var/www/html/web/sites/default/
```

Una vez añadido podemos observar las métricas del servicio memcached durante las visitas al portal drupal

watch "echo stats | nc 172.19.0.4 11211"


## Creación de multisite drupal

docker exec -ti my_drupal8_project_mariadb /bin/bash -c "mysql -u root -ppassword <<-EOSQL
CREATE DATABASE IF NOT EXISTS newsite;
GRANT ALL ON newsite.* TO 'drupal'@'%' ;
EOSQL"

docker exec -ti my_drupal8_project_php /bin/bash -c "cd /var/www/html/web/sites; mkdir newsite; cp default/default.settings.php newsite/settings.hp; chmod 644 newsite/settings.php"

echo "127.0.0.1 newsite.docker.localhost" | sudo tee -a /etc/hosts

docker cp sites.php my_drupal8_project_php:/var/www/html/web/sites/