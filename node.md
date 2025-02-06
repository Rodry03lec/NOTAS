## TODO EL TRABAJO CON NODE

#### PASO 1 Para inicializar el node.js **package.json**
```
npm init
```
#### PASO 2 en crear el index.js src/index.js
```
src/index.js
```
#### PASO 3 Instalacion del express
```
npm install express
```
#### PASO 4 Instalar el nodemon a nivel general, Si solo es necesario
```
npm install -g nodemon
```
#### Verificar o modificar el **package.json**
```
    "scripts": {
        "dev" : "nodemon src/index.js"
    },
```

### PARA REALIZAR LA PRUEBA CORRESPONDIENTE en **src/index.js**

```
const express = require("express");

//instanciamos en app
const app = express();

//para relizar la prueba si esta en funcionalidad
app.get("/", (req, res)=>{
    return res.json({
        mensaje: "Hola desde la ruta principal"
    });
});

//llamamos aqui, e iniciamos el servidor
app.listen(3000, function(){
    console.log("Servido iniciado en http://127.0.0.1:3000");
});
```
## CONTROLADOR creamos una carpeta en **src/controllers**
**producto.controller.js**
```
//para las funciones CRUD

function funcListar(req, respuesta){

}

function funcGuardar(req, respuesta){

}

function funcMostrar(req, respuesta){

}

function funcModificar(req, respuesta){

}

function funcEliminar(req, respuesta){

}

//No te olvides siempre exportar para importar en otros lugares

module.exports={
    funcListar,
    funcGuardar,
    funcMostrar,
    funcModificar,
    funcEliminar
}
```

## RUTAS creamos una carpeta en **src/routes**
**index.js**
```
//importamos express para la utilizacion
const express = require("express");

//importamos el controlador
const productoControlador = require("./../controllers/producto.controller"); 

//para realizar las rutas importamos el Router
const rutas = express.Router();

//realizamos las rutas conrrespondientes conectando con el controlador
rutas.get("/producto", productoControlador.funcListar);
rutas.post("/producto", productoControlador.funcGuardar);
rutas.get("/producto/:id", productoControlador.funcMostrar);
rutas.put("/producto:/id", productoControlador.funcModificar);
rutas.delete("/producto:id", productoControlador.funcEliminar);

//siempre exportar
module.exports = rutas;
```

## En la ruta principal **src/index.js**
```
//importamos las rutas
const rutas = require("./routes/index");  

//para que devuelva tipo JSON
app.use(express.json());

//habilitamos las rutas
app.use("/api", rutas);

```
## BASE DE DATOS conexion **src/database**
### instalar mongoose 
```
npm install mongoose
```
### Conectar a la base de datos MongoDB con mongoose
**index.js**
```
//importamos lo que requerimos mongoose
const mongose_db = require("mongoose");

//conectamos a la base de datos
mongose_db.connect("mongodb://127.0.0.1:27017/node_api_rest1");

//exportamos la conexion 
module.exports = mongose_db;
```
### Ahora en la misma carpeta de **src/database**
**Producto.js**
```
const mongoose_conect = require("./index");

//instanciamos que campos tedra
const producto = mongoose_conect.model("producto",{
    nombre: String,
    precio: Number,
    estado: Boolean,
    stock: Number,
    descripcion: String,
    imagen: String
}); 

//exportamos 
module.exports = producto;
```
## Editar en el controlador **producto.controller.js**
```
//importamos la instancia creado de producto
const Producto = require("./../database/Producto");

async function funcListar(req, respuesta){
    //listamos los productos
    const productos = await Producto.find({});
    //retornamos de tipo json
    return respuesta.json(productos);
}

async function funcGuardar(req, respuesta){
    //aqui es donde recivimos los datos
    const datos = req.body;  
    //instanciamos 
    const producto = new Producto(datos);
    //guardamos el producto
    await producto.save();

    return respuesta.json({mensaje: "Producto añadido con éxito ! "});

}
```

#### REALIZAR LAS PRUEBAS CORRESPONDIENTES