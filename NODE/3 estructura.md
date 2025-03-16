# ESTRUCTURA DE CARPETAS
## 1. Crear las rutas en src/routes
```
src/routes/index.js o rutas.js o nombre que desees o puedes separar
```
## 2. En "src/routes/rutas.js" 
```
//importamos
import express from "express";
import authControlador from "./../controlador/auth.controlador";


const Router = express.Router();

// http://127.0.0.1:3000/api/v1/auth/login

Router.post("/auth/login", authControlador.funLogin );
Router.post("/auth/registrar", authControlador.funRegistrar);
Router.get("/auth/perfil", authControlador.funPerfil);
Router.post("/auth/logout", authControlador.funLogout);

export default Router;
```

## 3. Debes crear el controlador "src/controlador/auth.controlador.js"
```
//aqui lo podemos llamar al modelo ya creado

//crear las funciones pero siempre exportando
export default {
    //req = para optener datos del navegador
    //res = es por parte del servidor
    funLogin(req, res){
        res.json({mensaje:"logueando"});
    },
    funRegistrar(req, res){
        res.json({mensaje:"Opteniendo el perfil"});
    },
    funPerfil(req, res){
        return res.json({mensaje: "hola desde funcion perfil"});
    },
    funLogout(req, res){
        res.json({mensaje:"Saliendo"});
    }
};
```

## 4. En el archivo principal index.js debemos llamar a las rutas o importar
***En "src/index.js"***
```
//importamos
import express from "express";
import urlRutas from "./routes/rutas";

//declaramos app
const app = express();

//aqui para la ruta base
app.use("/api/v1", urlRutas);


//aqui vamos a escuchar el puerto iniciar el servidor
app.listen(3000, function(){
    console.log("Servidor iniciado en http://127.0.0.1:3000");
});
```
