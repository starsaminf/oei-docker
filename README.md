# Oei-Docker

# Creando el entorno de desarrollo Back End

### Clonar el repositorio
```bash
git clone git@github.com:starsaminf/oei-docker.git
```

### Crear la red oei-net en docker
```bash
docker create network  oei-net
```

### Construir la imagen
```bash
cd oei-docker/back
docker build -t oei:laravel .
```
###Revisamos el Gatawey que nos asigno
```bash
docker inspect oei-net
```

### Construir el contenedor
Remplazar **/srv/htdocs/lenguas/ideback/** por la ruta del back end.


```bash
docker run --name oei-laravel  \
-e 'TIMEZONE=America/La_Paz'  \
-d -p 8084:80 \
--network oei-net \
--restart=always \
--ip 172.19.0.2 \
-v /srv/htdocs/lenguas/ideback/:/var/www/html  \
oei:laravel
```

###Configurando lenguas.dev en /etc/hosts

```bash
sudo nano /etc/hosts
```
##Agregamos en la parte final la ip que asignamos en el paso anterior
```bash
172.19.0.2      lenguas.dev
```

Con esta configuracion podremos ingresar desde el navegador lenguas.dev

#Entorno de desarrollo de la BD

##Construimos la imagen
```bash
cd oei-docker/db
docker build -t oei:db .
CREATE DATABASE identidad;
```

##Construimos el contenedor
```bash
docker run -d \
--network oei-net \
--name oei-db \
--ip 172.19.0.3 \
--restart=always \
-v /srv/htdocs/lenguas/data/bd:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root oei:db 
```
##Nos conectamos al contenedor
```bash
docker exec -it oei-db /bin/bash
mysql --user=root --password=root 
```
##Copiamos estas lineas 
```mysql
CREATE DATABASE identidad;

CREATE USER 'identidad'@'%' IDENTIFIED BY 'identidad';

GRANT ALL PRIVILEGES  ON identidad.* to 'identidad'@'%';

GRANT SELECT ON dclo.* TO identidad@'%';
```
###PhpMyAdmin
```bash
docker run --name oei-phpmyadmin \
-d -p 8085:80 \
--network oei-net \
--restart=always \
--ip 172.19.0.4 \
-e PMA_HOST=oei-db \
-e PMA_PORT=3306 \
phpmyadmin/phpmyadmin
```
##Agregamos en la parte final la ip que asignamos en el paso anterior
```bash
172.19.0.4      phpmyadminl.dev
```

Las credenciales son las mismas de mas arriba
```bash
usuario  root 
password root

usuario  identidad 
password identidad
```

###Restauramos la BD que se envio al correo


###Laravel  y BD
```bash
docker exec -it oei-laravel /bin/bash
cd public
cp .env.example .env
```
####Cambiar las credenciales de la BD
```bash
vi .env 
```
Con "sc + : wq" guardamos y salimos

####Remplazar las credenciales de la BD por
```bash
DB_CONNECTION=mysql
DB_HOST=oei-db
DB_PORT=3306
DB_DATABASE=identidad
DB_USERNAME=identidad
DB_PASSWORD=identidad
```
####Instalamos las dependencias
```bash
composer install
php artisan key:generate
php artisan jwt:secre
```
##Importamos la BD backup.sql que se nos envio al correo

###Front

Vamos donde esta nuestro front

```bash
cd /srv/htdocs/lenguas/idefront/
```
Levantamos con docker
```bash
docker run -it  \
--network oei-net \
--restart=always \
--ip 172.19.0.5 \
-w /opt -v $(pwd):/opt \
-p 4200:4200 \
alexsuch/angular-cli:1.0.0-beta.22-ubuntu ng \
serve --host 0.0.0.0
```













