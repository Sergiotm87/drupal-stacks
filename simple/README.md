# Instrucciones taller

- Levantar stack
   - Crear imagenes
   - Levantar servicios
   - Añadir entradas a /etc/hosts (opcional)

```shell
make up
```

- Comprobar los servicios levantados

```shell
docker-compose ps
```

Acceder a la url 'portainer.docker.localhost'

- Comprobar logs
```shell
docker-compose logs
docker-compose logs -f php
```
- Información de un servicio

```shell
docker inspect my_drupal8_project_nginx
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' my_drupal8_project_nginx

docker inspect my_drupal8_project_nginx | jq .[0] | jq keys
docker inspect my_drupal8_project_nginx | jq .[0].State
docker inspect my_drupal8_project_nginx | jq -r '.[].NetworkSettings.Networks[].IPAddress'
```

- Bootstrap drupal
   - creación de projecto drupal con composer

```shell
make drupal-bootstrap
```

- Acceder al site principal y realizar instalación
    - Comprobar valores para la base de datos en el fichero '.env'

Acceder a la url 'drupal.docker.localhost'