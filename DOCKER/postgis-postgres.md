# POSTGIS POSTGRES 
## Estructura
```
├── docker-compose.yml
└── init/
    └── 01-postgis.sql
```

### docker-compose-dev.yml
```
services:
  postgres_postgis:
    container_name: postgres_postgis
    image: postgis/postgis:16-3.4
    restart: always
    environment:
      POSTGRES_DB: gisdb
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgis_data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@gmail.com
      PGADMIN_DEFAULT_PASSWORD: password
    ports:
      - "5050:80"
    depends_on:
      - postgres_postgis

volumes:
  postgis_data:

```
### init/postgis.sql
```
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS postgis_topology;

```

### Ejecutamos el comando y ya
```
docker compose -f docker-compose-dev.yml up -d --build
```