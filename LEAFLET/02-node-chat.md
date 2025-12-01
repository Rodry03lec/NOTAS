# DOCKER CHAT
## En src/chatbot.js
```
function conectarwhasapp(){
    console.log("Hola mundo desde node.js ACTUALIZADO");
}

conectarwhasapp();
```
## iniciamos node con:
### Asi se creara el package.json
```
npm init -y
```
## Creamos el archivo *Dockerfile*
```
# Usar una imagen oficial en node js 
FROM node:20

#Establecer el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiar package*.json
COPY package*.json ./

# Instalar los paquetes o las dependencias de node.js
RUN npm install

# Copiar todo el codigo fuente
COPY . .

#para exponer puerto por el cual va correr la app
EXPOSE 3000

# Define el comando para correr la app

CMD ["node", "src/chatbot.js"]
```
## Para ejectutar el Dockerfile
### Para crear una imagen del proyecto
```
docker build -t node-chat:v2.0 .
```
### Para ejecutar el Dockerfile El v2.0 es una etiqueta
```
docker run node-chat:v2.0
```