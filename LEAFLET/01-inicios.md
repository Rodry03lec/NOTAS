# DOCKER
## Para buscar 
```
docker search mysql
docker search postgres
```
## IMAGENES
```
docker pull myslq:5.7
docker pull postgres -- esto te descarga la ultima version

docker image ls -- listar las imagenes
docker image rm codigo_imagen
```

## CONTENEDORES
### mysql
```
docker run -e MYSQL_ROOT_PASSWORD=root1 -p 3307:3306 mysql:5.7
```
### Con una base de datos
```
docker run -e MYSQL_ROOT_PASSWORD=root1 -e  MYSQL_DATABASE=mibd -p 3308:3306 mysql:5.7
```
### Listar los contenedores activos
```
docker container ls 
```
### Listar todos de los contenedor
```
docker container ls -a
```

### Para parar contenedor
```
docker container stop 37f15fa04aed
```
### Para eliminar contenedor
```
docker container rm 37f15fa04aed
```
### Para levantar o poner en marcha los contenedores
```
docker container start 37f15fa04aed
```
### Para eliminar todos los contenedores
```
docker container prune
```
