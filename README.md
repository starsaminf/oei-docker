# Oei-Docker



## Paso inicial para el BackEnd

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

### Construir el container
Remplazar **/srv/htdocs/lenguas/ideback/** por la ruta del backend

```bash
docker run --name oei-laravel  \
-e 'TIMEZONE=America/La_Paz'  \
-d -p 8084:80 \
--network oei-net \
--restart=always \
-v /srv/htdocs/lenguas/ideback/:/var/www/html  \
oei:laravel
```