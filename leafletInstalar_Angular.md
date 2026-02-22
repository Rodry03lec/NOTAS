# LEAFLET
## Instalacion de leaflet
### Librería JS
```
npm install leaflet
```
### Tipos TypeScript (muy importante en Angular)
#### Esto evita errores de TypeScript.
```
npm install --save-dev @types/leaflet
```
## Agregar estilos de Leaflet (MUY IMPORTANTE)
## En angular.json
#### Sin esto el mapa queda en blanco.
```
"styles": [
  "node_modules/leaflet/dist/leaflet.css",
  "src/styles.css"
]
```
