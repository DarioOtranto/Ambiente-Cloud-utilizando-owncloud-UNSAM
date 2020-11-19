# Ambiente-Cloud-utilizando-owncloud-UNSAM

_El presente proyecto constituye un TP final correspondiente a la materia ADMINISTRACION DE SISTEMAS GNU/LINUX Y VIRTUALIZACION-C-ELE78, UNSAM, segundo cuatrimestre del peor año de la historia, el 2020. El presente consiste en la instalación y configuración de un container de docker compose correspondiente a la creación de un ambiente Cloud con owncloud._

## 1) Creamos directorio del proyecto:

```
$ mkdir owncloud-docker-server
$ cd owncloud-docker-server
```
## 2) Copiamos docker-compose.yml del repo GitHub (owncloud):
```
$ wget https://raw.githubusercontent.com/owncloud/docs/master/modules/admin_manual/examples/installation/docker/docker-compose.yml
```
# 3) Editamos el archivo para personalizarlo un poco:
`
version: '3' **# aquí ponemos la versión más actualizada, por omisión es la 2.1**

volumes:
#  files:
#    driver: local
  mysql:
    driver: local
  backup:
    driver: local
  redis:
    driver: local

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    restart: always
    ports:
      - ${HTTP_PORT}:8080
    depends_on:
      - db
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=db
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - /home/negro-server/nube:/mnt/data # aquí seteamos la carpeta donde queremos que se guarden los archivos

  db:
    image: webhippie/mariadb:latest
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=owncloud
      - MARIADB_USERNAME=owncloud
      - MARIADB_PASSWORD=owncloud
      - MARIADB_DATABASE=owncloud
      - MARIADB_MAX_ALLOWED_PACKET=128M
      - MARIADB_INNODB_LOG_FILE_SIZE=64M
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - mysql:/var/lib/mysql
      - backup:/var/lib/backup

  redis:
    image: webhippie/redis:latest
    restart: always
    environment:
      - REDIS_DATABASES=1
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - redis:/var/lib/redis
``
# Creamos el archivo de configuración del ambiente:
cat << EOF > .env
OWNCLOUD_VERSION=10.5
OWNCLOUD_DOMAIN=localhost:8080
ADMIN_USERNAME="nuestro_usuario"
ADMIN_PASSWORD="nuestro_pass"
HTTP_PORT=8080
EOF

# Levantamos el container:

$ docker-compose up -d

Cuando se complete el proceso ejecutamos docker-compose ps y verificamos que todos los contenedores se hayan iniciado correctamente. Si todo funciona óptimo, deberíamos esperar lo siguiente:

Name                              Command                     State   Ports
__________________________________________________________________________________________
ownclouddockerserver_db_1         … /bin/s6-svscan /etc/s6    Up      3306/tcp
ownclouddockerserver_owncloud_1   … /usr/bin/owncloud server  Up      0.0.0.0:8080->8080/tcp
ownclouddockerserver_redis_1      … /bin/s6-svscan /etc/s6    Up      6379/tcp

La base de datos, ownCloud y los contenedores de Redis se están ejecutando. Sólo se puede acceder a ownCloud a través del puerto 8080 en la máquina host.

# Volúmenes

Todos los archivos almacenados en esta configuración están contenidos en volúmenes Docker, en lugar de un árbol del sistema de archivos físico. Para verlos ejecutamos:

$ docker volume ls | grep owncloud-docker-server

DRIVER              VOLUME NAME
local               owncloud-docker-server_backup
local               owncloud-docker-server_mysql
local               owncloud-docker-server_redis

# Actualización de ownCloud en Docker:

1) Vamos al directorio del proyecto:

$ ~/owncloud-docker-server 

2) Ponemos ownCloud en modo de mantenimiento:

$ docker-compose exec owncloud occ maintenance:mode --on

3) Creamos una copia de seguridad en caso de que algo salga mal durante el proceso de actualización:

$ docker-compose exec db backup

4) Cerramos los contenedores:

$ docker-compose down

NOTA: Aunque todos los datos importantes persisten después docker-compose down; docker-compose up -d, hay ciertos detalles que se pierden, por ejemplo, las aplicaciones predeterminadas pueden volver a aparecer después de desinstalarlas.

5) Editamos el archivo de configuración de entorno .env para cambiar el número de versión:

$ vim .env

OWNCLOUD_VERSION=10.5
OWNCLOUD_DOMAIN=localhost:8080
ADMIN_USERNAME="nuestro_usuario"
ADMIN_PASSWORD="nuestro_pass"
HTTP_PORT=8080

NOTA: dentro de la carpeta del proyecto podemos utilizar SED para automatizar el proceso con: 

$ sed -i 's/^OWNCLOUD_VERSION=.*$/OWNCLOUD_VERSION=<newVersion>/' /compose/*/.env
$ cat .env # para comprobar

6) Levantamos el container: 

$ docker-compose up -d

# Iniciamos sesión

En cualquier navegador tipeamos http://localhost:8080 y accedemos a la pantalla de inicio de sesión de ownCloud.

