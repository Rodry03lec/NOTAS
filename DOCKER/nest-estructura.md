## Estructura de carpetas
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

```
```