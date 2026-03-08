# NEST
## Configurar Inicialmente
### Estructura de carpetas
#### *Estrcuturalo como las carpetas correspondientes*
```
src/
├── common/              # Globales: guards, interceptors, filters, decorators
│   ├── decorators/
│   ├── filters/
│   └── middleware/
├── config/              # Configuración de variables de entorno y validación
├── database/            # Migraciones y seeders (si usas TypeORM/Prisma)
├── modules/             # El corazón de la app
│   ├── auth/            # Módulo de autenticación
│   │   ├── dto/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.module.ts
│   └── users/           # Módulo de usuarios
│       ├── domain/      # (Opcional) Entidades puras
│       ├── dto/         # Data Transfer Objects
│       ├── interfaces/
│       ├── users.controller.ts
│       ├── users.service.ts
│       └── users.module.ts
├── app.module.ts        # Módulo raíz que orquesta todo
└── main.ts              # Punto de entrada de la aplicación
```
## Para configurar los iconos
### *settings.json*
```javascript
{
    "material-icon-theme.activeIconPack": "nest"
}
```

## Para habilitar global el .env
### Instalar 
#### *Para .env global*
```javascript
npm i --save @nestjs/config
```
### Configurar
#### *En app.module.ts* dentro de imports
```javascript
imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    })
  ]
```
### Ahora crear .env y cambia el puerto 

## Prefix Global
### *En main.ts*
```
app.setGlobalPrefix('api');
```
# COMO EJEMPLO CREAR MANUALMENTE
### *En modules/auth*
#### *auth.module.ts*
```javascript
import { Module } from "@nestjs/common";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";

@Module({
    controllers:[AuthController],  // Aqui son los controladorres
    providers:[AuthService]  // Aqui son los 
})
export class AuthModule{

}
```
#### *auth.controller.ts*
```javascript
import { Controller, Get } from "@nestjs/common";
import { AuthService } from "./auth.service";

@Controller("auth")
export class AuthController{

    constructor(private AuthServicio: AuthService){}

    @Get()
    funListar(){
        return this.AuthServicio.authUsuario();
    }
}
```
#### *auth.service.ts*
```javascript
import { Injectable } from "@nestjs/common";

@Injectable()
export class AuthService{


    authUsuario(){
        return [ "Autenticacion", "Lista"]; 
    }
}
```
### Comandos:
### Crear un modulo
```javascript
nest g module modules/mapa --no-spec
```
### Crear un controlador 
```javascript
nest g co modules/mapa --no-spec
```
### Crear un servicio
```javascript
nest g s modules/mapa --no-spec
```
### O pueden usar lo que es para resource
#### porque ya crea: Controller, Service, Module, DTO, Entities
```javascript
nest g resource modules/mapa --no-spec
```


