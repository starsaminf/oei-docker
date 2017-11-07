# Oei-Docker Lenguas

### Clonar el repositorio
```bash
git clone git@github.com:starsaminf/oei-docker.git
```

#Creando el entorno de desarrollo BackEnd (Api)

###Crear la red oei-net en docker
```bash
docker network create oei-net
```

###Revisamos el Gatawey que se nos asigno, en este caso se asigno: 172.18.0.1
```bash
docker inspect oei-net | grep "Gateway" | awk -F: '{print $2}'
```
###Construir la imagen para laravel
```bash
cd oei-docker/back
docker build -t oei:api .
```
###Volvemos a la raiz del proyecto
```bash
cd ../../
```
###Construir el contenedor, usando la ip siguiente del Gatawey que se nos asigno, a modo de ejemplo seria 172.18.0.2
Remplazamos $(pwd)/ideback/ por la rata del ideback

```bash
docker run --name oei-api  \
-e 'TIMEZONE=America/La_Paz'  \
--network oei-net \
--restart=always \
--ip 172.19.0.2 \
-v $(pwd)/ideback/:/var/www/html  \
-d oei:api
```
###Editamos el archivo /etc/hosts

###Agregamos la ip del paso anterior  

```bash
echo "172.19.0.2 lenguas-api.dev" |  sudo tee --append  /etc/hosts
```
###Instalamos composer y las dependencias
```bash
docker exec -it oei-api /bin/bash
cd public/
composer install
php artisan key:generate
php artisan jwt:secre
```
Para salir control + d 

###Volvemos a la raiz del proyecto
```bash
cd ../../
```

#Entorno de desarrollo de la BD

##Construimos la imagen
```bash
cd oei-docker/db
docker build -t oei:db .
```
###Volvemos a la raiz del proyecto

```bash
cd ../../
```

##Construimos el contenedor, de igual modo asignamos una ip, para el ejemplo asignamos 172.19.0.3
```bash
docker run -d \
--network oei-net \
--name oei-db \
--ip 172.19.0.3 \
--restart=always \
-v $(pwd)/mysql_data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root oei:db 
```

##Conectamos con el contenedor
```bash
docker exec -it oei-db /bin/bash
mysql --user=root --password=root 
```

##Copiamos estas lineas 
```mysql
CREATE DATABASE identidad;
CREATE DATABASE dclo;

CREATE USER 'identidad'@'%' IDENTIFIED BY 'identidad';

GRANT ALL PRIVILEGES  ON identidad.* to 'identidad'@'%';
GRANT ALL PRIVILEGES  ON dclo.* to 'identidad'@'%';

GRANT SELECT ON dclo.* TO identidad@'%';
```
Apretamos control + d  2 veces


###PhpMyAdmin, cambiamos la ip, para el ejemplo usamos 172.19.0.4

```bash
docker run --name oei-phpmyadmin \
-d \
--network oei-net \
--restart=always \
--ip 172.19.0.4 \
-e PMA_HOST=oei-db \
-e PMA_PORT=3306 \
phpmyadmin/phpmyadmin

```

###Editamos el archivo /etc/hosts
###Agregamos la ip de phpmyadmin 

```bash
echo "172.19.0.4 lenguas-myadmin.dev" |  sudo tee --append  /etc/hosts
```
Las credenciales para la bae de datos

```bash
usuario  root 
password root

usuario  identidad 
password identidad
```

###Restauramos la BD que se envio al correo

Entramos desde el navegador a:  

```html
lenguas-myadmin.dev
```
Mostrara la pantalla de inicio a phpmyadmin

###Link Laravel - BD - Admin - User 
Desde nuestro IDE favorito editamos las credenciales del .env si es que es fuera necesario de:


####(Opcional) Remplazar las credenciales de la BD dentro de ideback .env
```bash
DB_CONNECTION=mysql
DB_HOST=oei-db
DB_PORT=3306
DB_DATABASE=identidad
DB_USERNAME=identidad
DB_PASSWORD=identidad
```

###Front Usuario

Vamos donde esta nuestro front

```bash
cd oei-docker/front/
docker build -t oei:lenguas .
cd ../../
```

```bash
cd idefront/
```

Instalamos las dependencias para angular, "$(pwd)" remplazamos por la ruta donde estan los archivos del front
```bash
docker run --rm -it -v $(pwd):/usr/src/app oei:front npm install 
```

###Levantamos con docker
```bash
docker run -d  \
--network oei-net \
--restart=always \
--ip 172.19.0.5  -p 4200:4200 \
-w /opt -v $(pwd):/opt \
--name lenguas-user \
oei:front ng \
serve --host 0.0.0.0 --port 80
     
```

###Logs en otra consola para el front
```bash
docker logs -f lenguas-user
```

###Editamos el archivo /etc/hosts
###Agregamos la ip de lenguas-user 

```bash
echo "172.19.0.5 lenguas.dev" |  sudo tee --append  /etc/hosts
```

###Front Admin

Instalamos las dependencias para angular, "$(pwd)" remplazamos por la rutal del admin front

```bash
docker run --rm \
-it -v $(pwd):/usr/src/app \
oei:front npm install 
```

Levantamos con docker, cambiamos la ip y "$(pwd)" remplazamos por la rutal del admin front

```bash
docker run -d  \
--network oei-net \
--restart=always \
--ip 172.19.0.6  \
-w /opt -v $(pwd):/opt \
--name  lenguas-admin \
oei:front ng \
serve --host 0.0.0.0 --port 80

```

###Logs
Para ver la consola

```bash
docker logs -f lenguas-user
```

###Editamos el archivo /etc/hosts
###Agregamos la ip de lenguas-admin 

```bash
echo "172.19.0.6 lenguas-admin.dev" |  sudo tee --append  /etc/hosts
```

###Archivos 
Restablecemos los archivos del storage, ejecutamos en la consola de la maquina

```bash
cp -rv oei-docker/files/ ideback/storage/app/
```

###Credenciales 
```bash
repositorios@oei.bo
ABCabc123
```
####Nota

Modificar los archivos config.ts de **idefront** y **admin**, editar las lineas de IDENTIDAD_URL de ambos archivos, debera quedar como se muestra a continuacion.

####Idefront

```javascript
..
  public static readonly IDENTIDAD_URL:string = 'http://lenguas.dev/api/v1';
  ..
```
####Admin

```javascript
..
   public static readonly IDENTIDAD_URL:string = 'http://lenguas.dev/api/v1';
..
```

En **ideback**

Modificar **ideback/config/cors.php**.

La linea 15 y agregar al array **lenguas.dev** y **lenguas-admin.dev**

Quedando el archivo

```php
'allowedOrigins' => ['http://localhost','http://localhost:4200','http://localhost:4100','http://localhost:5000','http://10.0.0.50:4200','http://10.0.1.18:4200','http://lenguas.dev','http://lenguas-admin.dev'],
```

En los comandos para lanzar los contenedores hay:

```bash
--restart=always 
```
Este comando se usa para iniciar automaticamente el contenedor.


Cada ves que se este desarrollando en angular, para ver los logs.

```bash
docker logs -f lenguas-user
```
o 

```bash
docker logs -f lenguas-admin
```

####Pendiente
Si tienes tiempo (lector) un script que automatice todo esto seria genial, ya me da flojera xD 

###Great 

Ya esta todo listo para desarrollo.


