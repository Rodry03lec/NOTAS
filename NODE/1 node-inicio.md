# NOTAS PARA NODE JS
## 1.Para iniciar con node.js 
***Complementas todo lo que te pide o "--y"***
```
npm init
```
## 2. Para instalar las dependencias de BABEL solo para proceso de desarrollo
```
npm i @babel/cli @babel/node @babel/core @babel/preset-env -D
```
***@babel/cli: Poder ejecutar interfas de linea de comandos***

***@babel/node: para poder utilizar con node***

***@babel/core: para las librerias***

***@babel/preset-env -D: es una forma de configurar mas rapido***

## 3. En la raiz del proyecto .babelrc
```
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets":{
                    "node":true
                }
            }
        ]
    ]
}
```

## 4. Instalar express
```
npm install express -y
```
## 5. Creamos un directorio src/index.js
***Para realizar las pruebas correspondientes***
```
import express from "express";

//declaramos app
const app = express();

//declaramos una ruta
app.get("/", function(req, res){
    return res.send({mensaje: "Bienvenido a node js"});
});

//aqui vamos a escuchar el puerto iniciar el servidor
app.listen(3000, function(){
    console.log("Servidor iniciado en http://127.0.0.1:3000");
});
```

## 6. En el packaje.json
***Aqui es donde configuramos para desarrollo build y start***
```
"scripts": {
    "dev": "npx --exec babel-node src/index.js",
    "build": "babel src/ --out-dir dist",
    "start": "node dist/index.js"
},
```
## 6. Instalamos nodemon en nuestro sistema o servidor

```
npm install -g nodemon
```
***Y cambiar en packaje.json***
```
"dev": "nodemon --exec babel-node src/index.js",
```

