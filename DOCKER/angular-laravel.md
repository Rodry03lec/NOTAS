# ANGULAR Y LARAVEL
## Crea un estructura:
## Es para crear inicialmente cuando no tienes un proyecto
```
mi-proyecto/
├── docker/
│   ├── php/
│   │   ├── Dockerfile.inicial
│   ├── angular/
│   │   ├── Dockerfile.inicial
│   └── nginx/
├── docker-compose.inicial.yml    # Para instalación inicial
```
### -> docker-compose.inicial.yml
```

services:
  # Serivcio temporal para crear laravel
  laravel-instalar:
    build:
      context: .
      dockerfile: docker/php/Dockerfile.inicial
    container_name: laravel-instalar
    volumes:
      - ./:/var/www
    working_dir: /var/www
    command: >
      bash -c "
      if [ ! -d 'backend' ]; then
        echo 'Instalando Laravel...'
        composer create-project laravel/laravel backend
      else
        echo 'Laravel ya existe'
      fi
      "

  
  # Servicio temporal para crear angular
  angular-instalar:
    build:
      context: .
      dockerfile: docker/angular/Dockerfile.inicial
    container_name: angular-instalar
    volumes:
      - ./:/app
    working_dir: /app
    command: >
      bash -c "
      if [ ! -d 'frontend' ]; then
        echo 'NOTA front: Instalando Angular'
        ng new frontend --skip-git
        echo 'NOTA front: Angular instalado'
      echo
        echo 'Angular ya existe'
      fi
      "
```

### -> docker/php/Dockerfile.inicial
```
FROM php:8.2-fpm

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# Instalar extencionas de php
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Instalar composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

CMD ["php-fpm"]
```
### -> docker/angular/Dockerfile.inicial
```
FROM node:22

WORKDIR /app

# Instalar angular CLI globalmente
RUN npm install -g @angular/cli

CMD ["tail", "-f", "/dev/null"]
```

### -> Para crear los comandos para verificar cada una
```
# Instalar laravel
docker compose -f docker-compose.inicial.yml run --rm angular-instalar 
# Instalar angular
docker compose -f docker-compose.inicial.yml run --rm laravel-instalar 

# Limpiar los contenedores temporales
docker compose -f docker-compose.inicial.yml down
```

# Para dar los permisos a cada carpeta
```
sudo chown -R $USER:$USER frontend backend
```

# MODO DESARROLLO

## >> LARAVEL <<
### Crear en docker/php/Dockerfile.dev
```
FROM php:8.2-fpm

# Instalar las dependencias
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    nano

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Instalar extenciones PHP
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Instalar composer 
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configurar usuario para evitar problemas de permisos
RUN useradd -G www-data,root -u 1000 -d /home/devuser devuser

RUN mkdir -p /home/devuser/.composer && \
    chown -R devuser:devuser /home/devuser

WORKDIR /var/www/backend

USER devuser

EXPOSE 9000

CMD ["php-fpm"]
```
### Nginx para Laravel
### crea en docker/nginx/laravel.conf
```
server {
    listen 80;
    index index.php index.html;
    root /var/www/backend/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php_dev:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

## >> ANGULAR <<
### Crea en docker/angular/Dockerfile.dev
```
FROM node:22

WORKDIR /app

# Instalar Angular CLI
RUN npm install -g @angular/cli

EXPOSE 4200

# Copiar entrypoint
COPY --chown=node:node docker/angular/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```


### Adicionalmente para la entrada en angular 
### crea en docker/angular/entrypoint.sh
```
#!/bin/bash
set -e

cd /app

echo "Verificando dependencias de Node..."

if [ ! -d "node_modules" ]; then
  echo "Instalando dependencias..."
  npm install
fi

echo "Iniciando Angular en modo desarrollo..."
exec ng serve --host 0.0.0.0 --port 4200 --poll=2000

```
## En la raiz crea :
## docker-compose.dev.yml
```
services:
  mysql_dev:
    image: mysql:8.0
    container_name: mysql_dev
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel_dev
      MYSQL_USER: user_dev
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data_dev:/var/lib/mysql
    ports:
      - "3307:3306"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  # PHP-FPM
  php_dev:
    build:
      context: .
      dockerfile: docker/php/Dockerfile.dev
    container_name: php_dev
    restart: unless-stopped
    volumes:
      - ./backend:/var/www/backend
    depends_on:
      mysql_dev:
        condition: service_healthy
    networks:
      - app-network
    environment:
      - DB_HOST=mysql_dev
      - DB_DATABASE=laravel_dev
      - DB_USERNAME=user_dev
      - DB_PASSWORD=password
    
  #Nginx para laravel
  nginx-backend_dev:
    image: nginx:alpine
    container_name: nginx-backend_dev
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - ./backend:/var/www/backend
      - ./docker/nginx/laravel.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php_dev
    networks:
      - app-network

  # Angular
  angular-dev:
    build:
      context: .
      dockerfile: docker/angular/Dockerfile.dev
    container_name: angular-dev
    restart: unless-stopped
    volumes:
      - ./frontend:/app
    ports:
      - "4200:4200"
    networks:
      - app-network
    environment:
      - CHOKIDAR_USEPOLLING=true
      - CI=true
    stdin_open: true
    tty: true

networks:
  app-network:

volumes:
  mysql_data_dev:
```
## Comandos para la ejecucion
### Para solo levantar angular
```
docker compose -f docker-compose.dev.yml build --no-cache angular-dev
```
### Verifica logs de angular
```
docker logs -f angular-dev
```
### Para levantar todos los servicions
```
docker compose -f docker-compose.dev.yml up -d
```
### Para bajar todos los servicios
```
docker compose -f docker-compose.dev.yml down
```

## Para ingresar tanto al Backend y Frontend
```
docker exec -it php_dev bash

docker exec -it angular-dev bash
```
```