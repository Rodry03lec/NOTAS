# POSTGIS POSTGRES 
## Estructura
```
├── docker-compose.yml
└── init/
    └── 01-postgis.sql
```

### docker-compose-dev.yml
```
# Define los servicios (contenedores) que Docker va a crear
services:

  # SERVICIO: PostgreSQL + PostGIS
  postgres_postgis:

    # Nombre fijo del contenedor
    container_name: postgres_postgis

    # Imagen oficial que incluye PostgreSQL + extensión espacial PostGIS
    image: postgis/postgis:17-3.5

    # Reinicia automáticamente si el contenedor falla o el servidor reinicia
    restart: always

    # Variables de entorno usadas por la imagen oficial de PostgreSQL
    environment:
      POSTGRES_DB: postgisdb        # Base de datos creada automáticamente
      POSTGRES_USER: usuario        # Usuario administrador
      POSTGRES_PASSWORD: password   # Contraseña del usuario

    # Mapeo de puertos
    # Tu PC usará el puerto 5433 → PostgreSQL interno usa 5432
    ports:
      - "5433:5432"

    # Volúmenes para persistencia y automatización
    volumes:

      # Guarda físicamente los datos de PostgreSQL
      # Evita perder información al borrar el contenedor
      - postgis_data:/var/lib/postgresql/data

      # Monta la carpeta local ./init dentro del contenedor
      # Todo archivo aquí se ejecuta automáticamente al crear la BD
      - ./init:/docker-entrypoint-initdb.d

  # SERVICIO: pgAdmin (Cliente Web)
  pgadmin:

    # Nombre del contenedor
    container_name: gadmin

    # Imagen oficial de pgAdmin4
    image: dpage/pgadmin4

    # Reinicio automático
    restart: always

    # Credenciales para acceder al panel web de pgAdmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@gmail.com
      PGADMIN_DEFAULT_PASSWORD: password

    # Puerto de acceso desde el navegador
    ports:
      - "5050:80"

    # Indica que este servicio depende del contenedor PostgreSQL
    depends_on:
      - postgres_postgis


# DEFINICIÓN DE VOLÚMENES
volumes:
  # Volumen persistente administrado por Docker
  # Guarda los datos reales de PostgreSQL
  postgis_data:


```
### init/postgis.sql
```
-- Activa la extensión principal PostGIS
-- Permite usar tipos geometry y geography
CREATE EXTENSION IF NOT EXISTS postgis;

-- Activa el módulo de topología espacial
-- Usado para redes, validación geométrica y análisis avanzado
CREATE EXTENSION IF NOT EXISTS postgis_topology;


```

### Ejecutamos el comando y ya
```
docker compose -f docker-compose-dev.yml up -d --build
```

### Para entrar el bash
```
docker exec -it postgres_postgis bash
```