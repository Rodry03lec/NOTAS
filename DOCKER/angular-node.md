# ANGULAR Y NODE

## INSTALACION: desde cero node y angular

### proyecto/docker-compose-inicial.yml

```
services:
  backend-iniciar:
    image: node:22-alpine
    working_dir: /app
    volumes:
      - ./backend:/app
    command: >
      sh -c "
      if [ ! -f package.json ]; then
        echo '>>>> Inicializando backend Node <<<<'
        npm init -y
        npm install express
      else
        echo 'Backend Node ya existe'
      fi
      "


  frontend-iniciar:
    image: node:22-alpine
    working_dir: /app
    volumes:
      - ./frontend:/app
    stdin_open: true
    tty: true
    command: >
      sh -c "
      if [ ! -f angular.json ]; then
        echo '>>>> Inicializando Angular en el directorio actual <<<<'
        npm install -g @angular/cli
        npx -p @angular/cli@21 ng new frontend --directory .
      else
        echo 'Proyecto Angular ya existe'
      fi
      "
```

### Ejecutar lo siguiente puedes hacerlo por separado

```
docker compose -f docker-compose-inicial.yml run frontend-iniciar
docker compose -f docker-compose-inicial.yml run backend-iniciar
```
### Para dar los persisos a las carpetas
```
sudo chown -R $USER:$USER frontend backend
```

### Solo en el backend como es node se debe iniciar
#### backend/src/index.js
```
import express from 'express';

const app = express();
const API_PREFIX = '/api';

app.get('/', (req, res)=>{
  res.json({ message: 'Backend funcionando XD sdasad' });
});

app.listen(3000, ()=>{
  console.log("backend escuchando http://localhost:3000");
});
```

#### en backend .gitignore
```
/node_modules
.vscode/
.env
```

## DESARROLLO:
### Crea una carpeta
### docker->back->Dockerfile.dev

```
FROM node:22-alpine

WORKDIR /app

COPY backend/package*.json ./

RUN npm install

COPY backend .

EXPOSE 3000

CMD [ "npm", "run", "dev" ]
```

### docker->front/Dockerfile.dev

```
FROM node:22-alpine

WORKDIR /app

COPY frontend/package*.json ./

RUN  npm install

COPY frontend .

EXPOSE 4200

CMD [ "npm", "start", "--", "--host", "0.0.0.0" ]
```

### docker-compose-dev.yml

```
services:
  backend-dev:
    container_name: backend-dev
    build:
      context: .
      dockerfile: docker/back/Dockerfile.dev
    ports:
      - "3002:3000"
    volumes:
      - ./backend:/app
      - backend_node_modules:/app/node_modules

  frontend-dev:
    container_name: frontend-dev
    build:
      context: .
      dockerfile: docker/front/Dockerfile.dev
    ports:
      - "4202:4200"
    volumes:
      - ./frontend:/app
      - frontend_node_modules:/app/node_modules

volumes:
  backend_node_modules:
  frontend_node_modules:
```

### Ejecutamos los siguientes comandos
```
docker compose -f  docker-compose-dev.yml up --build -d
```

### Para la ejecutar comandos en el contenedor frontend y backend
```
docker exec -it frontend-dev sh
docker exec -it backend-dev sh
```
### Para ver los logs 
```
docker compose -f docker-compose-dev.yml logs -f   
docker compose -f docker-compose-dev.yml logs -f frontend-dev
docker compose -f docker-compose-dev.yml logs -f backend-dev
```
### Para ver los logs pero menos codigo con el NOMBRE DE CONTENEDOR
```
docker logs -f backend-dev
docker logs -f frontend-dev
```


