## Para restaurar base de datos con postgres
```
docker-compose down -v
docker-compose up -d
docker cp dump-db_capacitaciones_ca-202603220911 postgres_postgis:/dump.backup
docker exec -it postgres_postgis bash
pg_restore -U usuario -d postgisdb --no-owner --no-privileges /dump.backup
```
