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

# < ========================= >
## Importamos Leaflet en el componente
```
import { Component, AfterViewInit } from '@angular/core';
import * as L from 'leaflet';
```
#### *AfterViewInit :* Es un ciclo de vida de Angular que se ejecuta cuando el HTML del componente ya fue renderizado en el DOM.
#### Esto es importante porque Leaflet necesita que el contenedor del mapa ya exista físicamente en la vista.

#### _* as L_ : Importamos la librería Leaflet y la usamos con el alias L. Esto nos permite utilizar:  L.map() L.tileLayer()

## Implementamos el ciclo de vida
```
export class Mapa implements AfterViewInit {
```
#### Implementamos *AfterViewInit* porque el mapa no puede crearse antes de que el HTML exista.
#### Si intentamos crearlo antes, Angular aún no habrá renderizado el <div id="map">.

## Declaramos la instancia del mapa
```
private mapa!: L.Map;
```
#### mapa almacenará la instancia del mapa.
#### L.Map es el tipo proporcionado por Leaflet.
### El signo (!)signo de admiracion indica que la variable será inicializada más adelante.

## Ciclo de vida: ngAfterViewInit
```
ngAfterViewInit(): void {
  this.iniciarMapa();
}
```
#### Angular primero dibuja el HTML.
#### Se ejecuta ngAfterViewInit.
#### Recién podemos crear el mapa.

## Método iniciarMapa()
```
private iniciarMapa(): void {

  this.mapa = L.map('map', {
    center: [-16.5, -68.15],  // Coordenadas iniciales (Latitud, Longitud) -- bolivia
    zoom: 6,                 // Nivel de acercamiento
    zoomControl: false       // Activa o desactiva los botones + -
  });

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(this.mapa);
}
```
#### L.map('map') :  Busca el elemento HTML con id "map" y crea dentro de él un mapa interactivo.
#### ¿Qué hace tileLayer? :  Leaflet funciona mediante tiles (pequeñas imágenes).  Descarga múltiples imágenes y las ensambla para formar el mapa completo.  Aquí estamos usando las teselas del proyecto OpenStreetMap.
 
# < ========================= >

## Ahora nos vamos al html
```
<div class="relative h-screen w-full overflow-hidden bg-slate-100">
```
### *relative* : Define el contenedor como referencia para posicionamiento interno.
### *h-screen* : Altura = 100% del alto de la pantalla. Es equivalente a _height: 100vh;_  Permite que el mapa ocupe toda la vista.
### *w-full* :  Ancho = 100% del contenedor.
### *overflow-hidden* : Evita que elementos internos se desborden.  Nos va servir para agregar controles flotantes
### *bg-slate-100* : Color de fondo gris claro (solo visual, casi no se verá porque el mapa lo cubre).

## Contenedor del mapa
```
<div id="map" class="absolute inset-0 z-0"></div>
```
### *absolute* : Se posiciona respecto al contenedor padre (relative).
### *inset-0* : Es equivalente a:  top: 0; right: 0; bottom: 0; left: 0;  Hace que el mapa ocupe todo el espacio disponible.
### *z-0* :  Nivel de profundidad bajo. Nos servirá cuando agreguemos botones o elementos flotantes encima del mapa

# NOTA
## Hasta aqui debes tener lo siguiente
```
import { AfterViewInit, Component } from '@angular/core';

import * as L from 'leaflet';

@Component({
  selector: 'app-mapa',
  imports: [],
  templateUrl: './mapa.html',
  styleUrl: './mapa.scss',
})
export class Mapa implements AfterViewInit {

  private mapa!: L.Map;

  ngAfterViewInit(){
    this.iniciarMapa();
  }

  iniciarMapa(){
    this.mapa = L.map('mapa_id', {
      center: [-16.5, -68.15], // coordenadas iniciales (latitud y longitu) -- bolivia
      zoom: 6,
      zoomControl: true
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'OpenStreetMap'
    }).addTo(this.mapa);

  }

}
```
## El Html
```
<div class="relative h-screen w-full overflow-hidden bg-slate-500" >
  <div id="mapa_id" class="absolute inset-0 z-0"></div>
</div>
```


# 2 ::::::::::: Agregado de componentes
## Componente header
```
ng g c mapa/header --skip-tests
```
### header.ts
```
import { ToolbarModule } from 'primeng/toolbar';
import { ButtonModule } from 'primeng/button';

drawerVisible: boolean = false;   // Controla si el menú lateral móvil está abierto o cerrado.
```
### header.html
```

```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```


```
```