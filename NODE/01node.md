# INSTALACION DE NODE JS
## 1. iniciar node.js
```
npm init
```
## 2. Para instalar las dependencias de BABEL solo para proceso de desarrollo
```
npm i @babel/cli @babel/node @babel/core @babel/preset-env -D
```
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

## 4. Instalar express y morgan y para conectar postgres
```
npm i express morgan pg
```
## 5. Instalar, para que te digan los errores
```
npm i standart -D
```
## 6. en el packaje.json debes agregarg //ag
```
{
  "name": "01back_node",
  "version": "1.0.0",
  "main": "index.js",
  "type": "module", //ag
  "scripts": {
    "dev": "node --watch src/index.js", //ag
    "build": "babel src/ --out-dir dist", //ag
    "start": "node dist/index.js" //ag
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "devDependencies": {
    "@babel/cli": "^7.26.4",
    "@babel/core": "^7.26.10",
    "@babel/node": "^7.26.0",
    "@babel/preset-env": "^7.26.9",
    "standart": "^6.1.0"
  },
  "dependencies": {
    "express": "^4.21.2",
    "morgan": "^1.10.0",
    "pg": "^8.14.1"
  },
  "eslintConfig": {
    "extends": "standart" 
  } //ag
}
```
```
```
```
```
```
```
```
```
```
```