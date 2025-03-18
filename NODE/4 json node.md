# Para las peticiones
***Para aceptar peticiones en formato JSON (req.body) : antes de las rutas***

### Esto configurar en el archivo principal index.js
```
app.use(express.json());
```

## 1. Para realizar pruebas correspondientes
```
async funRegistrar(req, res) {
    //querys = /api/v1/auth/registrar?estado=true&mensaje=Hola
    //console.log(req.query);
    // params /api/v1/auth/registrar/:nombre?email=admin@gmail.com
    //en url auth/registrar/:nombre
    //console.log(req.params);
    //headers = /api/v1/auth/registrar
    //console.log(req.headers);
    //body
    //console.log(req.body);
    //return res.json({ mensaje: req.body });
    //para registrar un usuario

    console.log("DATOS ENTRANTES", req.body);

    let datos_usuario = req.body;
    //encriptar la contraseña
    datos_usuario.password = await bcrypt.hash(datos_usuario.password);

    console.log(datos_usuario);
},
```
## 2. Instalar "bcrypt"
```
npm install bcrypt
```

## 3. Para cifrar las contraseñas
``` 
console.log("DATOS ENTRANTES", req.body);

let datos_usuario = req.body;
//encriptar la contraseña
datos_usuario.password = await bcrypt.hash(datos_usuario.password, 12);
``` 
## 4. Instalar jsonwebtoken
```
npm install jsonwebtoken
```
## 5. Ejemplo Completo Auth
```
import bcrypt from "bcrypt"

//aqui lo podemos llamar al modelo ya creado
import models from "./../database/models"

//importamos jsonwebtoken
import jwt from "jsonwebtoken"

//crear las funciones pero siempre exportando
export default {
    //req = para optener datos del navegador
    //res = es por parte del servidor
    async funLogin(req, res) {
        //destructuracion
        const { email, password } = req.body;
        const usuario = await models.User.findOne({
            where: {
                email: email
            }
        });

        if(!usuario){
            return res.status(401).json(
                {mensaje: "Credenciales Incorrectas"}
            );
        }
        //verificar la contraseña
        let correcto = await bcrypt.compare(password, usuario.password);
        if(correcto){
            //jenerera JWT
            let payload = {
                id: usuario.id,
                email: usuario.email,
                time: new Date()
            }
            const token = jwt.sign(payload, "JST_SECRET", {
                expiresIn: 60*60,
            });
            return res.status(200).json({
                access_token: token,
                usuario: usuario,
                error: false
            });
        }else{
            return res.status(401).json({mensaje:"Credenciales Incorrectas"});
        }
    },
    async funRegistrar(req, res) {
        //querys = /api/v1/auth/registrar?estado=true&mensaje=Hola
        //console.log(req.query);
        // params /api/v1/auth/registrar/:nombre?email=admin@gmail.com
        //en url auth/registrar/:nombre
        //console.log(req.params);
        //headers = /api/v1/auth/registrar
        //console.log(req.headers);
        //body
        //console.log(req.body);
        //return res.json({ mensaje: req.body });
        //para registrar un usuario

        console.log("DATOS ENTRANTES", req.body);

        let datos_usuario = req.body;
        //encriptar la contraseña
        datos_usuario.password = await bcrypt.hash(datos_usuario.password, 12);

        const usuario = await models.User.create(datos_usuario);
        if(usuario.id)
            return res.status(201).json({mensaje:"usuario Registrado"});
        
        return res.status(422).json(
            {
                error:true,
                mensaje:"Error al registrar el usuario"
            }
        );
    },
    funPerfil(req, res) {
        return res.json({ mensaje: "hola desde funcion perfil" });
    },
    funLogout(req, res) {
        res.json({ mensaje: "Saliendo" });
    },
};
```