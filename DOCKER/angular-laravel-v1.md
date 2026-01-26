# ANGULAR - LARAVEL
## Para crear inicialmente cuando no tienes un proyecto

### Estructura de carpetas
```
mi-proyecto/
|-- docker/
│   |-- php/
│   │   |-- Dockerfile.inicial
│   |-- angular/
│   │   |-- Dockerfile.inicial
│   |-- nginx/
|-- docker-compose.inicial.yml    # Para instalación inicial
```

#### docker/php/Dockerfile.inicial
```
FROM php:8.2-cli

# Dependencias del sistema
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Extensiones PHP necesarias para Laravel
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www
```

#### docker/angular/Dockerfile.inicial
```
FROM node:22-alpine

WORKDIR /app

RUN npm install -g @angular/cli@21
```

#### docker-compose-inicial.yml
```
services:
  angular-inicial:
    container_name: angular-inicial
    build:
      context: .
      dockerfile: docker/angular/Dockerfile.inicial
    volumes:
      - ./:/app
    working_dir: /app
    command: |
      sh -c '
      if [ ! -f frontend/angular.json ]; then
        echo " Iniciando proyecto Angular en frontend/..."
        ng new frontend --directory frontend --skip-git
      else
        echo " Proyecto Angular ya existe en frontend/"
      fi
      '
  
  laravel-inicial:
    container_name: laravel-inicial
    build:
      context: .
      dockerfile: docker/php/Dockerfile.inicial
    volumes:
      - ./:/var/www
    working_dir: /var/www
    command: |
      sh -c '
      if [ ! -f backend/artisan ]; then
        echo " Iniciando proyecto Laravel..."
        composer create-project laravel/laravel backend
      else
        echo " Laravel ya existe en backend/"
      fi
      '
```

#### Comandos para crear y verificar cada una
```
docker compose -f docker-compose-inicial.yml run --rm angular-inicial
docker compose -f docker-compose-inicial.yml run --rm laravel-inicial

#limpiar los contenedores temporales
docker compose -f docker-compose-inicial.yml down
```

#### Para dar los permisos a cada carpeta de los proyectos
```
sudo chown -R $USER:$USER frontend backend
```

## MODO DE DESARROLLO
## Laravel
#### Crear en docker/php/Dockerfile.dev
```
FROM php:8.2-fpm

# Dependencias del sistema
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && docker-php-ext-install \
    pdo_mysql \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Usuario no root
RUN useradd -u 1000 -m -s /bin/bash -G www-data devuser

WORKDIR /var/www/backend

# Permisos Laravel
RUN chown -R devuser:www-data /var/www/backend

USER devuser

CMD ["php-fpm"]
```

#### Crear en docker/nginx/laravel.conf
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
        fastcgi_pass php_developer:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

## Angular
#### Crear en docker/angular/Dockerfile.dev
```
FROM node:20

WORKDIR /app

# Angular CLI
RUN npm install -g @angular/cli

EXPOSE 4200

CMD sh -c "rm -rf node_modules package-lock.json && npm install && ng serve --host 0.0.0.0 --port 4200 --poll=2000"
```

#### Crear docker-compose-dev.yml
```
services:
  mysql_developer:
    container_name: mysql_developer
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel_dev
      MYSQL_USER: user_dev
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data_dev:/var/lib/mysql
    ports:
      - "3309:3306"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10


  php_developer:
    container_name: php_developer
    build:
      context: .
      dockerfile: docker/php/Dockerfile.dev
    restart: unless-stopped
    volumes:
      - ./backend:/var/www/backend
    depends_on:
      mysql_developer:
        condition: service_healthy
    networks:
      - app-network


  # Nginx para laravel
  nginx_backend:
    container_name: nginx_backend
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ./backend:/var/www/backend
      - ./docker/nginx/laravel.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php_developer
    networks:
      - app-network


  angular_developer:
    container_name: angular_developer
    build:
      context: .
      dockerfile: docker/angular/Dockerfile.dev
    volumes:
      - ./frontend:/app
    ports:
      - "4300:4200"
    networks:
      - app-network
    environment:
      - NG_CLI_ANALYTICS=false


networks:
  app-network:

volumes:
  mysql_data_dev:
```

## Comandos para la ejecion de comandos
#### Para levantar todos los servicios
```
docker compose -f docker-compose-dev.yml up -d  --build
```
#### Para bajar los servicios
```
docker compose -f docker-compose-dev.yml down
```

#### Para ver los logs
```
docker compose -f docker-compose-dev.yml logs -f angular_developer
docker compose -f docker-compose-dev.yml logs -f php_developer

o

docker logs -f angular_developer
docker logs -f php_developer
```

#### Para ingresar tanto al backend y frontend
```
docker exec -it php_developer bash
docker exec -it angular_developer bash
```

#### Limpieza ocacional para que no haya muchas imagenes
```
docker image prune
```

