# README.md development-environment-react-php-mysql

# Descripción

Este repositorio se compone del esqueleto de un proyecto de React creado con `npx create-react-app` en donde se ha realizado limpieza de los archivos innecesarios.

También se incorpora un fichero `docker-compose.yml` que genera la arquitectura necesaria para facilitar el desarrollo de aplicaciones en donde se requiere un backend basado en PHP y MySQL. Por ejemplo, para elaborar una API que será consumida por la aplicación basada en React.

# Explicación del fichero `docker-compose.yml`

## Servicio de PHP

Este servicio actúa de endpoint y es accesible a través de *http://localhost* y el puerto que se haya especificado. Se trata de un servidor web Apache y PHP 8.

Toda la lógica de PHP deberá ir en la carpeta `backend` que se encuentra en la raíz del directorio de trabajo.

```
  # Backend service
  php:
    build:
      context: ./dockerfiles/php
      dockerfile: 8-apache-buster.yml
    ports:
      - ${PHP_ENDPOINT_LOCAL_PORT}:80
    volumes:
      - "./backend:/var/www/html"
```

## Servicio de Nginx

Este servicio levanta un servidor web Nginx accesible a través de la url *http://localhost* y el puerto que se haya especificado. Es el punto de acceso a la aplicación de React una vez que se haya construido con `npm run build` y ubicado el código compilado en el directorio `build`.

```
  # Frontend service
  nginx:
    image: nginx
    ports:
      - ${NGINX_LOCAL_PORT}:80
    volumes:
      - "./build:/usr/share/nginx/html"
```

## Servicio de base de datos

El servicio de base de datos lo compone un servidor MariaDB. Como herramienta adicional, para facilitar el acceso a través de una GUI, se ha incorporado PHPMyAdmin también como servicio.

```
# Database services
  mariadb:
    image: mariadb:10
    command: --max_allowed_packet=${DATABASE_MAX_ALLOWED_PACKET}
    environment:
      - MYSQL_ROOT_PASSWORD=${DATABASE_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DATABASE_NAME}
    volumes:
      - "./data:/var/lib/mysql"

  phpmyadmin:
    image: phpmyadmin
    ports:
      - ${PHPMYADMIN_LOCAL_PORT}:80
    environment: 
      - MYSQL_ROOT_PASSWORD=${DATABASE_ROOT_PASSWORD}
      - PMA_HOST=mariadb
    depends_on:
      - mariadb
```

# Requisitos

- Docker (≥ 20.10.6)
- Docker Compose (≥ 1.29.2)
- Git
- Node.js
- NPM

# Configuración y levantamiento del entorno de desarrollo

## Clonar el repositorio en local

```
git clone https://github.com/thebrokenbrain/development-environment-react-php-mysql.git
```

## Crear el fichero .env

Renombrar el fichero `.env.example` por `.env`, y configurar las variables de entorno que se necesitan. Por ejemplo:

```
# Variables de entorno para el servicio "php"
PHP_ENDPOINT_LOCAL_PORT=8000

# Variables de entorno para el servicio "nginx"
NGINX_LOCAL_PORT=8080

# Variables de entorno para el servicio "mariadb"
DATABASE_NAME=database
DATABASE_ROOT_PASSWORD=password
DATABASE_MAX_ALLOWED_PACKET=67108864 # 64MB

# Variables de entorno para el servicio "phpmyadmin"
PHPMYADMIN_LOCAL_PORT=9000
```

Las variables asociadas a los puertos (`PHP_ENDPOINT_LOCAL_PORT, NGINX_LOCAL_PORT, PHPMYADMIN_LOCAL_PORT`) deben ser únicas por cada entorno de desarrollo que se levante.

## Arrancar los servicios

Ejecutar el siguiente comando en la carpeta raíz del directorio de trabajo (en donde se encuentra el fichero `docker-compose.yml`)

```
docker-compose up -d
```

Después de ejecutar el comando anterior, para verificar que todos los servicios se encuentran levantados correctamente (`Up`), se puede ejecutar el siguiente comando en el directorio de trabajo:

```
docker-compose ps
```

Para preparar el entorno de desarrollo clona el repositorio y ejecuta el siguiente comando

```bash
docker-compose up -d
```

## Instalar las dependencias del proyecto de React

Por último ya solo queda instalar las dependencias y paquetes necesarios del proyecto de React.

Para hacer esto hay que ejecutar el siguiente comando desde la carpeta raíz del directorio de trabajo:

```
npm install
```

Tener en cuenta que el comando anterior leerá el fichero `package.json` que se ha descargado desde este repositorio. Si lo necesitas, puedes modificarlo y adaptarlo a tus necesidades.
