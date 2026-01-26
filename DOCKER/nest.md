# NEST 
## Desde cero el proyecto
## docker-compose-inicial.yml
```
services:
  backend-iniciar:
    image: node:22-alpine
    working_dir: /app
    volumes:
      - ./backend:/app
    stdin_open: true
    tty: true
    command: >
      sh -c "
      if [ ! -f package.json ]; then
        echo 'Instalando backend nest'
        npm i -g @nestjs/cli
        nest new backend --directory .
      else
        echo 'backend nest ya existe'
      fi
      "

  frontend-iniciar:
    image: node:22-alpine
    working_dir: /app
    volumes:
      - ./frontend:/app
    stdin_open: true
    tty: true
    command: sh -c "npm create vite@latest . -- --template vue-ts"
```
## Para iniciar
```
docker compose -f docker-compose-inicial.yml run backend-iniciar
docker compose -f docker-compose-inicial.yml run frontend-iniciar
```

## Permisos
```
sudo chown -R $USER:$USER frontend backend
```

## Docker/back/Dockerfile.dev
```
FROM node:22-alpine

WORKDIR /app

COPY backend/package*.json ./

RUN npm install

COPY backend .

EXPOSE 3000

CMD [ "npm", "run", "start:dev" ]
```

## Docker/front/Dockerfile.dev
```
FROM node:22-alpine

WORKDIR /app

COPY frontend/package*.json ./
RUN npm install

COPY frontend .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host"]
```

## docker-compose-dev.yml
```
services:
  backend-dev:
    container_name: backend-dev
    build:
      context: .
      dockerfile: docker/back/Dockerfile.dev
    ports:
      - "3005:3000"
    volumes:
      - ./backend:/app
      - backend_node_modules:/app/node_modules
    

  frontend-dev:
    container_name: frontend_dev
    build:
      context: .
      dockerfile: docker/front/Dockerfile.dev
    ports:
      - "5177:5173"
    volumes:
      - ./frontend:/app
      - frontend_node_modules:/app/node_modules

volumes:
  backend_node_modules:
  frontend_node_modules:
```
## Ejecutar
```
docker compose -f docker-compose-dev.yml logs -f
docker compose -f docker-compose-dev.yml logs -f frontend-dev
docker compose -f docker-compose-dev.yml logs -f backend-dev

```
