#  para base de datos "SEQUELIZE"
## 1. Instalar Sequelize 
### https://sequelize.org/docs/v6/getting-started/
```
npm install --save sequelize
```
## 2. Instalamos los drivers para trabajar con el gestor de base de datos
```
npm install --save pg pg-hstore # Postgres
npm install --save mysql2
npm install --save mariadb
npm install --save sqlite3
npm install --save tedious # Microsoft SQL Server
npm install --save oracledb # Oracle Database
```
## 3. Migraciones si es desde cero si no, no.
***Instalamos la interfas de liena de comandos***
```
npm install --save-dev sequelize-cli
```
## 4. para los archivos de configuracion para las migraciones
***En la raiz te creas un .sequelizerc para dar orden a nustras carpetas***
```
const path = require("path");

module.exports = {
    config: path.resolve("src/config", "database.json"), 
    "models-path": path.resolve("src/database", "models"),
    "seeders-path": path.resolve("src/database", "seeders"),
    "migrations-path": path.resolve("src/database", "migrations"),
};
```
## 5. Ejecutar el comando para que me lo cree las carpetas correspondientes
```
npx sequelize-cli init
```

## 6. SUBIR A GIT
***No te olvides que #.gitignore" para que no se suba todo el proyecto***

```
dist
node_modules
```

## 7. GENERAR MIGRACIONES
#### Generar un modelo + migraciones
```
npx sequelize-cli model:generate --name User --attributes nombre:string,email:string,password:string
```

## 8. Migrar la base de datos
```
npx sequelize-cli db:migrate
```