# HABILITAR DOCKER COMPOSE
### 1. Ejecuta los siguientes comandos para descargar e instalar Docker Compose:
```
# Descargar la última versión de Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Dar permisos de ejecución al binario
sudo chmod +x /usr/local/bin/docker-compose
```

### 2. Verificar la instalación
```
docker-compose --version
```
### 3. Crear un contenedor para base de datos mysql
***Debe tener el nombre creado en el proyecto "docker-compose.yml"***

```
version: '3.8'

services:
  mysql:
    image: mysql:8.0  # Usar la imagen oficial de MySQL 8.0
    container_name: mysql_contenedor 3306  # Nombre del contenedor
    environment:
      MYSQL_ROOT_PASSWORD: password  # Contraseña para el usuario root
      MYSQL_DATABASE: bd_node  # Nombre de la base de datos que se creará al iniciar
      MYSQL_USER: rodrigo  # Usuario de la base de datos
      MYSQL_PASSWORD: rodry  # Contraseña del usuario
    ports:
      - "3306:3306"  # Mapear el puerto 3306 del contenedor al puerto 3306 del host
    volumes:
      - mysql_data:/var/lib/mysql  # Volumen para persistir los datos de la base de datos
    networks:
      - mysql_network  # Red para conectar este servicio con otros (opcional)

volumes:
  mysql_data:  # Definir un volumen para persistir los datos de MySQL

networks:
  mysql_network:  # Definir una red personalizada (opcional)
``` 
***postgres***
```
version: '3.8'

services:
  postgres:
    image: postgres:15  # Usar la imagen oficial de PostgreSQL 15
    container_name: postgres_container  # Nombre del contenedor
    environment:
      POSTGRES_USER: my_user  # Usuario de la base de datos
      POSTGRES_PASSWORD: my_password  # Contraseña del usuario
      POSTGRES_DB: my_database  # Nombre de la base de datos que se creará al iniciar
    ports:
      - "5432:5432"  # Mapear el puerto 5432 del contenedor al puerto 5432 del host
    volumes:
      - postgres_data:/var/lib/postgresql/data  # Volumen para persistir los datos de la base de datos
    networks:
      - postgres_network  # Red para conectar este servicio con otros (opcional)

volumes:
  postgres_data:  # Definir un volumen para persistir los datos de PostgreSQL

networks:
  postgres_network:  # Definir una red personalizada (opcional)
```
### 4. Ejecutar o hacer correr .yml
``` 
docker compose up -d
``` 

