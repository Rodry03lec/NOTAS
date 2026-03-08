# Guía de Clase — Componente Mapa (Angular + Leaflet + PrimeNG)
## Tecnologías utilizadas

- **Angular 21 — Framework principal
- **Leaflet** — Librería de mapas interactivos
- **PrimeNG** — Componentes visuales (Tabs, Button, Card, Drawer...)
- **Tailwind CSS** — Estilos utilitarios

---

## Paso 1 — Solo el Mapa

> **Concepto:** Contenedor base + inicialización de Leaflet con `AfterViewInit`

### `mapa.ts`

```typescript
import { Component, ElementRef, inject, NgZone, signal, ViewChild, AfterViewInit } from '@angular/core';
// Component → Permite definir un componente en Angular
// ElementRef → Permite acceder directamente a un elemento del DOM
// inject → Nueva forma moderna de inyectar dependencias
// NgZone → Controla el cambio de detección de Angular (mejora rendimiento)
// signal → Sistema reactivo moderno de Angular para manejar estado
// ViewChild → Permite obtener referencia de un elemento del HTML
// AfterViewInit → Ciclo de vida que se ejecuta cuando el HTML ya está renderizado

// CommonModule → Permite usar directivas comunes como ngIf, ngFor en el HTML
import { CommonModule } from '@angular/common';

// Importamos toda la librería Leaflet con el alias "L"
import * as L from 'leaflet';

// CONFIGURACIÓN DEL ICONO POR DEFECTO DE LEAFLET
// Leaflet normalmente busca sus iconos dentro de node_modules,
// lo cual en Angular suele generar errores de rutas.

// Eliminamos la función interna que busca automáticamente los iconos
delete (L.Icon.Default.prototype as any)._getIconUrl;

// Definimos manualmente las rutas del icono
L.Icon.Default.mergeOptions({

  // Icono normal
  iconUrl: 'leaflet/icono.png',

  // Icono para pantallas retina
  iconRetinaUrl: 'leaflet/icono.png',

  // Quitamos la sombra del marcador
  shadowUrl: '',

  // Tamaño del icono
  iconSize: [40, 40],

  // Punto exacto donde el icono toca el mapa
  iconAnchor: [20, 40],

  // Posición donde aparece el popup respecto al icono
  popupAnchor: [0, -35]
});



@Component({
  selector: 'app-mapa',
  imports: [CommonModule],
  templateUrl: './mapa.html',
  styleUrl: './mapa.scss',
})

// CLASE DEL COMPONENTE
// Implementamos AfterViewInit porque Leaflet necesita
// que el div del mapa exista antes de inicializarse
export class Mapa implements AfterViewInit {

  // Variable donde se guardará la instancia del mapa
  private mapa!: L.Map;

  // Inyectamos NgZone usando la nueva API de Angular
  private zone = inject(NgZone);

  // SIGNAL REACTIVO

  // Signal que guarda las coordenadas actuales
  // Cuando cambia su valor Angular actualiza la UI automáticamente
  coordenadas = signal({
    latitud: '-16.5000',
    longitud: '-68.1500'
  });

  // REFERENCIA AL HTML
  // ViewChild obtiene referencia al elemento del HTML
  // Ejemplo: <div #mapContainer></div>
  @ViewChild('mapContainer') mapContainer!: ElementRef;

  // CICLO DE VIDA
  // Se ejecuta cuando el HTML del componente ya fue renderizado
  // Este momento es ideal para iniciar Leaflet
  ngAfterViewInit(): void {
    this.iniciarMapa();
  }

  // FUNCIÓN QUE CREA EL MAPA
  private iniciarMapa(): void {

    // Creamos el mapa dentro del div con id "mapa_id"
    this.mapa = L.map('mapa_id', {
      center: [-16.5, -68.15], // Coordenadas iniciales del mapa (La Paz - Bolivia)
      zoom: 6, // Nivel de zoom inicial
      zoomControl: false // Ocultamos el control de zoom por defecto
    });

    // CAPA BASE DEL MAPA

    // TileLayer carga las imágenes del mapa desde OpenStreetMap
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      // Texto de atribución obligatorio por licencia
      attribution: '&copy; OpenStreetMap contributors',
      // Zoom máximo permitido
      maxZoom: 19
    }).addTo(this.mapa); // Agregamos la capa al mapa


    // EVENTO DEL MOUSE
    // runOutsideAngular evita que Angular ejecute
    // detección de cambios en cada movimiento del mouse
    // lo que mejora mucho el rendimiento
    this.zone.runOutsideAngular(() => {
      // Evento que se ejecuta cada vez que el mouse se mueve en el mapa
      this.mapa.on('mousemove', (e: L.LeafletMouseEvent) => {
        // Actualizamos las coordenadas usando el signal
        this.coordenadas.set({
          // latlng contiene las coordenadas del cursor
          latitud: e.latlng.lat.toFixed(4),
          // toFixed limita los decimales
          longitud: e.latlng.lng.toFixed(4)
        });


        console.log(
          'Latitud:',
          this.coordenadas().latitud,
          'Longitud:',
          this.coordenadas().longitud
        );
        
      });
    });


    // RESPONSIVE DEL MAPA
    // Cuando cambia el tamaño de la ventana
    // recalculamos el tamaño del mapa
    window.addEventListener('resize', () => {
      // invalidateSize fuerza a Leaflet a recalcular el tamaño
      this.mapa.invalidateSize();
    });
  }
}
```

### `mapa.html`

```html
<div class="relative h-screen w-full overflow-hidden bg-slate-100">

  <!-- 
    id="mapa_id"     → Leaflet lo busca por este ID
    absolute inset-0 → ocupa toda la pantalla
    z-0              → queda debajo de todo lo demás
    #mapContainer    → referencia para ViewChild en el .ts
  -->
  <div id="mapa_id" class="absolute inset-0 z-0" #mapContainer></div>

</div>
```

---

## Paso 2 — Agregamos el Header

> **Concepto:** Comunicación padre→hijo con `@Input` e hijo→padre con `@Output`

### `mapa.ts` — agregar

```typescript
import { Header } from './header/header';

// En imports del @Component:
imports: [CommonModule, Header],

// En la clase:
drawerVisible: boolean = false;
```

### `mapa.html` — agregar

```html
<!-- 
  [drawerVisible] → le enviamos el estado al header     (@Input)
  (openDrawer)    → escuchamos cuando pide abrir drawer (@Output)
-->
<app-header
  [drawerVisible]="drawerVisible"
  (openDrawer)="drawerVisible = true">
</app-header>
```

---
### header.ts
```typescript
// Importamos herramientas básicas de Angular
import { Component, EventEmitter, Input, Output } from '@angular/core';

import { ToolbarModule } from 'primeng/toolbar';  // Barra superior tipo toolbar
import { ButtonModule } from 'primeng/button';    // Botones estilizados
import { CommonModule } from '@angular/common';   // Directivas básicas de Angular (ngIf, ngFor, etc.)

@Component({
  selector: 'app-header',
  imports: [
    CommonModule,  // Permite usar directivas estructurales como *ngIf
    ToolbarModule, // Permite usar el componente de barra superior
    ButtonModule   // Permite usar botones de PrimeNG
  ],
  templateUrl: './header.html',
  styles: ``,
})

// Definimos la clase del componente
export class Header {

  // INPUT
  // Permite recibir datos desde el componente padre
  // En este caso recibe el estado de visibilidad del Drawer (panel lateral)
  @Input() drawerVisible!: boolean;

  // OUTPUT
  // Permite enviar eventos desde este componente hacia el componente padre
  // EventEmitter crea un evento personalizado
  @Output() openDrawer = new EventEmitter();

  // Método que se ejecuta cuando el usuario hace click en el botón del menú
  abrirMenu() {

    // Emitimos el evento hacia el componente padre
    // El componente padre escuchará este evento y abrirá el drawer
    this.openDrawer.emit();
  }
}

```

### header.html
```html
<!-- Contenedor del encabezado -->
<!-- Se posiciona de forma absoluta en la parte superior del visor -->
<header class="absolute top-0 left-0 right-0 z-50 p-2 md:p-4">

  <!-- Contenedor central para limitar el ancho máximo -->
  <div class="mx-auto max-w-7xl">

    <!-- Toolbar de PrimeNG -->
    <!-- Se usa como barra superior con efectos visuales -->
    <p-toolbar class="backdrop-blur-md bg-white/70 border border-white/20 shadow-xl rounded-2xl px-3 py-2">

      <!-- SECCIÓN IZQUIERDA DE LA TOOLBAR -->
      <div class="p-toolbar-group-start gap-2">

        <!-- Botón hamburguesa para abrir el menú lateral -->
        <!-- Solo visible en dispositivos móviles -->
        <p-button icon="pi pi-bars" [text]="true" severity="secondary" class="md:hidden" (onClick)="abrirMenu()" />

        <!-- Contenedor del logo y título -->
        <div class="flex items-center gap-2">

          <!-- Icono del visor -->
          <div class="bg-primary-600 p-2 rounded-xl">
            <!-- Icono de mapa usando PrimeIcons -->
            <i class="pi pi-map text-white text-lg"></i>
          </div>

          <!-- Título del visor -->
          <span class="text-lg font-black tracking-tighter uppercase">
            Geo
            <!-- Segunda parte del nombre con color diferente -->
            <span class="text-primary-600 font-light">
              Visor
            </span>
          </span>
        </div>
      </div>


      <!-- SECCIÓN DERECHA DE LA TOOLBAR -->
      <div class="p-toolbar-group-end gap-2">

        <!-- Botón de búsqueda -->
        <!-- Representa funcionalidad futura para buscar lugares -->
        <p-button
          icon="pi pi-search"
          [rounded]="true"
          severity="primary"
          class="h-9 w-9" />
      </div>
    </p-toolbar>
  </div>
</header>


```
---

## Paso 3 — Agregamos el Footer

> **Concepto:** Pasar datos en tiempo real con `signal()` a un componente hijo

### `mapa.ts` — agregar

```typescript
import { Footer } from './footer/footer';

// En imports del @Component:
imports: [CommonModule, Header, Footer],
```

### `mapa.html` — agregar

```html
<!-- 
  coordenadas() → llamamos al signal como función para leer su valor actual
-->
<app-footer
  [lat]="coordenadas().latitud"
  [lng]="coordenadas().longitud">
</app-footer>
```
---

### footer.ts
```typescript
// Input permite recibir datos desde un componente padre
import { Component, Input } from '@angular/core';

// Importamos CommonModule que incluye directivas básicas de Angular
// como *ngIf, *ngFor, etc.
import { CommonModule } from '@angular/common';

// Decorador que define la configuración del componente
@Component({
  selector: 'app-footer',

  // Módulos que necesita este componente
  imports: [CommonModule],
  templateUrl: './footer.html',
  styles: ``,
})

export class Footer {

  // Decorador Input
  // Permite que el componente padre envíe datos a este componente
  // Variable que recibirá la latitud desde el componente padre
  @Input() lat!: string;

  // Variable que recibirá la longitud desde el componente padre
  @Input() lng!: string;
}

```
### footer.html
```html
<!-- Contenedor principal del footer -->
<!-- Se posiciona de forma absoluta en la parte inferior del visor -->
<footer class="absolute bottom-3 left-1/2 -translate-x-1/2 z-30 hidden md:block">
  <!-- Caja que contiene la información de coordenadas -->
  <!-- Tiene efecto vidrio (glass effect) usando blur y transparencia -->
  <div class="bg-slate-900/80 backdrop-blur-md text-white px-5 py-2 rounded-full text-[10px] font-mono border border-white/10 shadow-2xl flex gap-4">

    <!-- Contenedor que agrupa las coordenadas -->
    <div class="flex gap-3">
      <!-- Latitud -->
      <span>
        <!-- Texto descriptivo -->
        LAT:
        <!-- Valor dinámico que viene desde el componente padre -->
        <!-- Angular lo renderiza usando interpolación -->
        <span class="text-emerald-400">
          {{ lat }}
        </span>
      </span>

      <!-- Longitud -->
      <span>
        <!-- Texto descriptivo -->
        LONG :
        <!-- Valor dinámico enviado desde el componente del mapa -->
        <span class="text-emerald-400">
          {{ lng }}
        </span>
      </span>
    </div>
  </div>
</footer>

```

---

## Paso 4 — Agregamos el Menú

> **Concepto:** `ng-template` reutilizable + panel desktop + drawer mobile

### `mapa.ts` — agregar

```typescript
import { TabsModule } from 'primeng/tabs';
import { CardModule } from 'primeng/card';
import { DrawerModule } from 'primeng/drawer';

// En imports del @Component:
imports: [..., TabsModule, CardModule, DrawerModule],

// En la clase:
activeTab: string = 'menu';
```

### `mapa.html` — agregar

```html
<!-- 
  PANEL LATERAL DESKTOP
  hidden md:block → solo visible en pantallas medianas o grandes
  ng-container    → inserta el template sin agregar un div extra al DOM
-->
<aside class="absolute z-40 top-24 left-6 w-80 lg:w-96 hidden md:block">
  <p-card styleClass="shadow-2xl border-none backdrop-blur-lg bg-white/90 rounded-2xl overflow-hidden">
    <ng-container *ngTemplateOutlet="menuContent"></ng-container>
  </p-card>
</aside>

<!-- 
  DRAWER MOBILE
  [(visible)] → two-way binding con drawerVisible
-->
<p-drawer
  [(visible)]="drawerVisible"
  header="Menú de Capas"
  [style]="{ width: '85vw', 'max-width': '350px' }">
  <ng-container *ngTemplateOutlet="menuContent"></ng-container>
</p-drawer>

<!-- 
  TEMPLATE REUTILIZABLE
  #menuContent → se usa tanto en el panel desktop como en el drawer mobile
  Evita duplicar el mismo HTML dos veces
-->
<ng-template #menuContent>
  <div class="p-4">

    <p-tabs [(value)]="activeTab">
      <p-tablist class="bg-transparent border-b border-slate-100">
        <p-tab value="menu" class="flex-1 font-bold text-sm">Menu</p-tab>
        <p-tab value="capas_almacenadas" class="flex-1 font-bold text-sm">Capas Almacenadas</p-tab>
      </p-tablist>
    </p-tabs>

    <div class="py-4 max-h-[60vh] overflow-y-auto">

      <!-- @if → nuevo flujo de control Angular 17+ (reemplaza *ngIf) -->
      @if (activeTab === 'menu') {
        <p>Aquí irá el menú</p>
      }

      @if (activeTab === 'capas_almacenadas') {
        <p>Aquí irán las capas almacenadas</p>
      }

    </div>
  </div>
</ng-template>
```

---

## Paso 5 — Agregamos los Mini Mapas (BaseMaps)

> **Concepto:** `@for` para iterar + `[ngClass]` para estilos condicionales

### `mapa.ts` — agregar

```typescript
import { TooltipModule } from 'primeng/tooltip';
import { BASE_MAPS_CONFIG } from '../config/tipos_mapa';

// En imports del @Component:
imports: [..., TooltipModule],

// En la clase:
readonly mapas_base = BASE_MAPS_CONFIG;
mapaSeleccionadoId: string = 'streets';
private mapaBaseActual?: L.TileLayer;

// En iniciarMapa() reemplazar capa fija por:
setTimeout(() => this.cambiarMapaBase(this.mapas_base[0]));

// Nuevo método:
cambiarMapaBase(config: any) {
  this.mapaSeleccionadoId = config.id;
  if (this.mapaBaseActual) {
    this.mapa.removeLayer(this.mapaBaseActual);
  }
  this.mapaBaseActual = L.tileLayer(config.url, {
    attribution: config.atributo,
    maxZoom: 19
  }).addTo(this.mapa);
}
```

### `mapa.html` — agregar

```html
<!-- 
  BASEMAPS — esquina inferior izquierda
  @for       → nuevo flujo Angular 17+ (reemplaza *ngFor)
  track m.id → optimización: Angular identifica cada elemento por su id
  [pTooltip] → muestra el nombre al hacer hover
  [ngClass]  → aplica estilos distintos si ese mapa está seleccionado
-->
<div class="absolute bottom-6 left-4 md:bottom-10 md:left-6 z-40">
  <div class="flex gap-3 p-2 bg-white/30 backdrop-blur-md rounded-2xl border border-white/40 shadow-2xl">

    @for (m of mapas_base; track m.id) {
      <div
        (click)="cambiarMapaBase(m)"
        class="relative group cursor-pointer transition-all duration-300"
        [pTooltip]="m.nombre"
        tooltipPosition="top">

        <div
          class="h-14 w-14 md:h-16 md:w-16 rounded-xl border-2 overflow-hidden transition-all shadow-lg group-hover:-translate-y-2"
          [ngClass]="{
            'border-primary-500 ring-4 ring-primary-500/20': mapaSeleccionadoId === m.id,
            'border-white': mapaSeleccionadoId !== m.id
          }">

          <img [src]="m.imagen" class="w-full h-full object-cover">

          <!-- Check solo en el mapa activo -->
          @if (mapaSeleccionadoId === m.id) {
            <div class="absolute inset-0 bg-primary-500/20 flex items-center justify-center">
              <i class="pi pi-check text-white bg-primary-500 rounded-full p-1 text-[8px]"></i>
            </div>
          }

        </div>
      </div>
    }

  </div>
</div>
```

---

## Paso 6 — Agregamos las Herramientas

> **Concepto:** Botones que llaman métodos del `.ts` para controlar Leaflet directamente

### `mapa.ts` — agregar

```typescript
import { ButtonModule } from 'primeng/button';

// En imports del @Component:
imports: [..., ButtonModule],

// Nuevos métodos:
  // =============================
  // CONTROLES DEL MAPA
  // =============================

  // Acercar el mapa
  acercar() {
    this.mapa.zoomIn();
  }

  // Alejar el mapa
  alejar() {
    this.mapa.zoomOut();
  }


  // =============================
  // UBICACIÓN DEL USUARIO
  // =============================

  miUbicacion() {

    // Activa la geolocalización del navegador para intentar obtener la posición del usuario
    this.mapa.locate({
      enableHighAccuracy: true // Solicita al navegador usar GPS o la mayor precisión posible
    });

    // Se ejecuta una sola vez cuando Leaflet logra obtener la ubicación del usuario
    this.mapa.once('locationfound', (e: any) => {
      // Mueve el mapa suavemente hasta las coordenadas encontradas y aplica zoom 20
      this.mapa.flyTo(e.latlng, 20);
      // Crea un marcador en las coordenadas obtenidas de la geolocalización
      L.marker(e.latlng)
        // Agrega el marcador al mapa actual
        .addTo(this.mapa)
        // Asocia un popup al marcador con el texto "Mi ubicación"
        .bindPopup("Mi ubicación")
        // Abre automáticamente el popup cuando se crea el marcador
        .openPopup();
    });

  }
```

### `mapa.html` — agregar

```html
  <!-- CONTROLES MAPA -->

  <div class="absolute right-4 bottom-10 md:right-6 md:bottom-12 z-40 flex flex-col gap-3">

    <!-- Zoom + -->
    <p-button
      icon="pi pi-plus"
      [rounded]="true"
      severity="info"
      (onClick)="acercar()"
      class="h-8 w-8 md:h-10 md:w-10">
    </p-button>

    <!-- Zoom - -->
    <p-button
      icon="pi pi-minus"
      [rounded]="true"
      severity="info"
      (onClick)="alejar()"
      class="h-8 w-8 md:h-10 md:w-10">
    </p-button>

    <!-- Ubicación -->
    <p-button
      icon="pi pi-compass"
      [rounded]="true"
      severity="info"
      (onClick)="miUbicacion()"
      class="h-8 w-8 md:h-10 md:w-10">
    </p-button>

  </div>
```

---

## Resumen — Qué agrega cada paso


| Paso | Qué agrega        | Concepto Angular clave              |
|------|-------------------|-------------------------------------|
| 1    | Div del mapa      | `AfterViewInit`, `ViewChild`         |
| 2    | Header            | `@Input`, `@Output`, Event binding  |
| 3    | Footer            | `signal()`, Property binding        |
| 4    | Menú              | `ng-template`, `@if`, two-way binding |
| 5    | Mini mapas        | `@for`, `[ngClass]`, `[pTooltip]`   |
| 6    | Herramientas      | `(onClick)`, métodos del componente |

---

## Conceptos clave de Angular usados

| Concepto | Sintaxis | Para qué sirve |
|----------|----------|----------------|
| Property binding | `[propiedad]="valor"` | Enviar datos al hijo |
| Event binding | `(evento)="metodo()"` | Escuchar eventos del hijo |
| Two-way binding | `[(valor)]="variable"` | Sincronizar dato en ambas direcciones |


### Activar environments
```
ng generate environments
```
### En envorontments
```javascript
  export const environment = {
    production: true, // y false en developer
    apiUrl: 'http://localhost:3000/api'
  };
```
### Creamos un servicio mapa servicio
#### ng g s servicio/mapa.service
```javascript
  //declarmaos la base url
  urlBase = environment.apiUrl;

  //inyectamos el http cliente
  http = inject(HttpClient);

  listarEscuelas(){
    return this.http.get(`${this.urlBase}/mapa/listar`);
  }
```
### en nest.
### Convertir una geometría de la base de datos a formato GeoJSON.
```javascript
  select
      id,
      nombre,
      direccion,
      ST_AsGeoJSON(geom) as geom
  from public.unidad_educativa;
```
## No me funciona por el caso de los cors en main.ts
```javascript
  app.enableCors({
    origin: ['http://localhost:4200'],
  });
```
