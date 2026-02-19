# Instalacion de Docker
## Pasos para la instalacion con la documentacion oficial de docker
### 1. *Actualizacion de repositorios*
```
sudo apt update
```

### 2. *Herramientas necesarias.*
#### - *ca-certificates* : permite conexiones HTTPS seguras
#### - *_curl_* : descargar archivos desde internet
```
sudo apt install ca-certificates curl
```


### 3. *Crear carpeta de llaves*
#### - *Linux guarda firmas digitales.*
```
sudo install -m 0755 -d /etc/apt/keyrings
```


### 4. *Descargar clave oficial Docker*
```
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
```


¿Qué es una clave GPG?
Es como un sello oficial.
Cuando instales Docker, Debian verificará:
Si la firma no coincide -> instalación bloqueada.
-> Previene malware.


### 5. Permisos de lectura
```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
#### - *APT necesita leer esa firma.*

### 6. Registrar el repositorio Docker
#### - *Agregar el repositorio a las fuentes de Apt*
```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### 7. Actualizar repositorios nuevamente
```
sudo apt update
```

### 8. Recién aquí instalas Docker real
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 9. Verificar la instalacion
```
sudo systemctl status docker

sudo systemctl start docker
```
### 10. Para no estar entrando como sudoo algo
```
sudo usermod -aG docker $USER
```

### 11. Reinciamos el equipo
