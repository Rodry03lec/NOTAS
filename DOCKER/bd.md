# BASE DE DATOS

## MYSQL
### POR TERMINAL
#### Para crear una imagen - contenedor
```
docker run --name dbmysql_admin -e MYSQL_ROOT_PASSWORD=rootpass -p 3331:3306 mysql:latest
```
#### Acceder desde una terminal
```
docker exec -it dbmysql_admin bash

mysql -u root -p
```
#### Para ver en phpmyadmin la interfaz grafica
```
docker run --name dbmysql_adminphpmy --link dbmysql_admin:db -p 9991:80 phpmyadmin/phpmyadmin
```
### POR docker-compose.yml
#### Primero se debe crear el archivo docker-compose.yml
```
services:
  dbmysql2:  # Nombre del servicios
    image: mysql:latest # imagen de donde va descargar
    container_name: dbmysql2 # el nombre del contenedor
    environment:  
      MYSQL_ROOT_PASSWORD: rootpass # la contraseña
    ports:
      - "3332:3306" # el puerto

    # Para la forma fisica
    volumes:
      - ./mysqldata:/var/lib/mysql  # Carpeta fisica donde se almacenara
```
#### Para levantar el contenedor
```
docker compose up
```
## POSTGRES
### POR TERMINAL
```
docker run --name dbpostgres1 -e POSTGRES_PASSWORD=rootpass -p 5444:5432 postgres:latest
```
### POR docker-compose.yml
#### Primero se debe crear el archivos de docker-compose.yml
```
services:
  postgresdb2:
    image: postgres:latest
    container_name: postgresdb2
    restart: always
    environment:
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: rootpass
    ports:
      - "5449:5432"
    volumes:
      #- pg_data:/var/lib/postgresql/data  # esto hasta la version 17 -
      - pg_data:/var/lib/postgresql   # esto la version 18 +

volumes:
  pg_data:
```
#### NOTA:
```
restart: no        # (por defecto) no se reinicia
restart: on-failure  # solo si el proceso falla
restart: always    # SIEMPRE se reinicia
restart: unless-stopped # siempre, excepto si tú lo paras
```

### PARA ACCEDER DESDE UNA TERMINAL
```
docker exec -it dbpostgres1 bash 

psql -U postgres
```


## COMANDOS docker-compose.yml
### Para ver que se esta ejecutando 
```
docker compose up
```
### Para ejecutar en segundo plano
```
docker compose up -d
```
### down: Detiene y elimina contenedores
### -v elimina volúmenes asociados
```
docker compose down -v
```

### Eliminar todos los volumenes puede eliminar uno por uno pero
### Tener cuidado
```
docker volume prune
```

### Para que docker quede como recien instalado
```
docker system prune -a --volumes
```