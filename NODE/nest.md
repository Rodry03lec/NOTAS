## NEST.JS
### Configuración Inicial
```
1. F1
2. workspaces json
3. y crear lo siguiente:
{
    "material-icon-theme.activeIconPack": "nest"
}
```

### En el .env 
#### Instalar lo siguiente
```
npm install @nestjs/config
```
### En app.module.ts
```
import { ConfigModule } from '@nestjs/config';
ConfigModule.forRoot({
  isGlobal: true,
}),
```

## Comandos para generar en NEST
### Genera siempre el el modulo primero
```
nest generate module admin/usuario --no-spec
nest generate controller admin/usuario --no-spec
nest generate service admin/usuario --no-spec
```
```
