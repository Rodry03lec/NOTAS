## Para dejar como recien isntalado docker
### Ver qué tienes actualmente
```
docker ps -a
docker images
docker volume ls
docker network ls
```

### Detener todos los contenedores 
```
docker stop $(docker ps -aq)
```

### Eliminar todos los contenedores
```
docker rm $(docker ps -aq)
```

### Eliminar todas las imágenes
```
docker rmi $(docker images -q)
```

### Eliminar TODOS los volúmenes (lo que tú quieres) 
```
docker volume rm $(docker volume ls -q)
```

### Eliminar redes no usadas
```
docker network prune -f
```
