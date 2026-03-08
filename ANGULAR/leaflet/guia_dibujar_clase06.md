
### Primero crear un servicio donde pintara mapas
```
ng g s servicios/geometria.service --skip-tests
```

### En el servicio lo que debo hacer es lo siguiente:
```javascript
import { Injectable } from '@angular/core';
import * as L from 'leaflet';

/*
  Interface que define la estructura mínima que debe tener cada elemento
  que será enviado al servicio para dibujar en el mapa.

  geom: contiene la geometría en formato GeoJSON (normalmente viene desde la base de datos)
  [clave: string]: any -> permite que el objeto tenga cualquier otra propiedad adicional
  (nombre, id, dirección, etc.) que luego se mostrará en el popup.
*/
interface ElementoGeometria {
  geom: string;
  [clave: string]: any;
}

/*
  Interface que define el resultado que el servicio devolverá al componente.

  capa   -> contiene todas las geometrías dibujadas dentro de un LayerGroup
  bounds -> contiene los límites geográficos de todas las geometrías
            para poder centrar el mapa usando fitBounds()
*/
interface ResultadoDibujo {
  capa: L.LayerGroup;
  limites: L.LatLngBounds;
}

/*
  Decorador de Angular que indica que esta clase es un servicio
  y puede ser inyectado en cualquier componente.

  providedIn: 'root' significa que existe una única instancia
  del servicio en toda la aplicación.
*/
@Injectable({
  providedIn: 'root',
})
export class GeometriaService {

  /*
    Método principal del servicio.

    Su función es recibir un conjunto de datos con geometrías
    y convertirlos en capas de Leaflet listas para mostrarse en el mapa.

    datos            -> lista de objetos con geometría y propiedades
    opcionesPunto    -> configuración visual para puntos (icono, tamaño, etc.)
    opcionesPoligono -> configuración visual para polígonos (color, opacidad, etc.)
  */
  dibujarGeometrias(datos: ElementoGeometria[], opcionesPunto?: any, opcionesPoligono?: any ): ResultadoDibujo {

    // Se crea un LayerGroup nuevo cada vez que se ejecuta el método.
    // Este grupo contendrá todas las geometrías dibujadas.
    const capa   = L.layerGroup();

    // Variable que almacenará los límites geográficos de todas las geometrías.
    // Esto permitirá luego hacer map.fitBounds().
    const limites = L.latLngBounds([]);

    // Recorremos cada elemento recibido desde el backend
    datos.forEach((elemento: ElementoGeometria) => {

      // Si el elemento no tiene geometría, se ignora
      if (!elemento.geom) return;

      let geometria: any;

      /*
        Intentamos convertir la geometría a objeto JSON.

        Algunas veces el backend envía el GeoJSON como string,
        por lo que necesitamos convertirlo usando JSON.parse().
      */
      try {
        geometria = typeof elemento.geom === 'string'
          ? JSON.parse(elemento.geom)
          : elemento.geom;
      } catch {

        // Si el JSON es inválido, se muestra advertencia y se omite el elemento
        console.warn('Geometría inválida, se omite el elemento:', elemento.geom);
        return;
      }

      // Obtenemos el tipo de geometría (Point, Polygon, MultiPolygon)
      const tipoGeometria = geometria.type;

      // Coordenadas de la geometría
      const coordenadas   = geometria.coordinates;

      // Variable donde se guardará la capa creada
      let layer: L.Layer | undefined;

      // ─────────── POINT ───────────────────────────────────────────
      // Si la geometría es un punto
      if (tipoGeometria === 'Point') {

        // GeoJSON usa formato [longitud, latitud]
        const [longitud, latitud] = coordenadas;

        /*
          Si se definieron opciones con icono personalizado
          se crea un marcador con imagen.
        */
        if (opcionesPunto?.iconUrl) {

          layer = L.marker([latitud, longitud], {
            icon: L.icon({
              iconUrl:     opcionesPunto.iconUrl,
              iconSize:    opcionesPunto.iconSize    ?? [20, 20],
              iconAnchor:  opcionesPunto.iconAnchor  ?? [12, 20],
              popupAnchor: opcionesPunto.popupAnchor ?? [0, -35]
            })
          });

        } else {

          /*
            Si no se define icono se dibuja un CircleMarker
            (un punto circular en el mapa)
          */
          layer = L.circleMarker([latitud, longitud], {
            radius:      8,
            color:       '#1565C0',
            fillColor:   '#1E88E5',
            fillOpacity: 0.85,
            weight:      2
          });
        }

        // Extendemos los límites del mapa con este punto
        limites.extend([latitud, longitud]);
      }

      // ─────────── POLYGON ─────────────────────────────────────────
      // Si la geometría es un polígono
      if (tipoGeometria === 'Polygon') {

        /*
          Convertimos las coordenadas de GeoJSON

          GeoJSON:
          [longitud, latitud]

          Leaflet:
          [latitud, longitud]
        */
        const verticesPoligono: [number, number][] =
          coordenadas[0].map((c: number[]) => [c[1], c[0]]);

        // Se crea el polígono con las opciones visuales
        layer = L.polygon(verticesPoligono, {
          color:       opcionesPoligono?.color           ?? 'blue',
          fillColor:   opcionesPoligono?.colorRelleno    ?? '#3388ff',
          fillOpacity: opcionesPoligono?.opacidadRelleno ?? 0.2,
          weight:      opcionesPoligono?.grosor          ?? 2
        });

        // Extendemos los límites con cada vértice
        verticesPoligono.forEach(v => limites.extend(v));
      }

      // ─────────── MULTIPOLYGON ────────────────────────────────────
      // Si la geometría contiene varios polígonos
      if (tipoGeometria === 'MultiPolygon') {

        const multipoligono = coordenadas.map((poligono: any) =>
          poligono[0].map((c: number[]) => [c[1], c[0]])
        );

        layer = L.polygon(multipoligono, {
          color:       opcionesPoligono?.color           ?? 'blue',
          fillColor:   opcionesPoligono?.colorRelleno    ?? '#3388ff',
          fillOpacity: opcionesPoligono?.opacidadRelleno ?? 0.2,
          weight:      opcionesPoligono?.grosor          ?? 2
        });

        // Se extienden los límites con todos los vértices
        multipoligono.flat().forEach((v: any) => limites.extend(v));
      }

      // ─────────── POPUP + HOVER ───────────────────────────────────
      // Si la capa fue creada correctamente
      if (layer) {

        // Se crea el popup con los datos del elemento
        layer.bindPopup(this.construirPopup(elemento));

        /*
          Evento cuando el mouse pasa sobre la geometría
          Se abre el popup y se resalta el polígono
        */
        layer.on('mouseover', (evento) => {
          const objetivo = evento.target;
          objetivo.openPopup();

          if (typeof objetivo.setStyle === 'function') {
            objetivo.setStyle({ weight: 4, fillOpacity: 0.5 });
          }
        });

        /*
          Evento cuando el mouse sale de la geometría
          Se cierra el popup y vuelve al estilo original
        */
        layer.on('mouseout', (evento) => {
          const objetivo = evento.target;
          objetivo.closePopup();

          if (typeof objetivo.setStyle === 'function') {
            objetivo.setStyle({
              weight:      opcionesPoligono?.grosor          ?? 2,
              fillOpacity: opcionesPoligono?.opacidadRelleno ?? 0.2
            });
          }
        });

        // Finalmente se agrega la geometría al grupo de capas
        capa.addLayer(layer);
      }

    });

    /*
      Se retorna al componente:

      capa   -> todas las geometrías listas para agregar al mapa
      bounds -> límites geográficos para poder centrar el mapa
    */
    return { capa, limites: limites };
  }

  /*
    Método privado que construye el contenido HTML del popup.

    Recibe el objeto completo del elemento y genera
    una tabla con sus propiedades.
  */
  private construirPopup(elemento: ElementoGeometria): string {

    // Campos que no se mostrarán en el popup
    const camposExcluidos = new Set(['geom']);

    // Se recorren las propiedades del objeto
    const filas = Object.entries(elemento)
      .filter(([clave]) => !camposExcluidos.has(clave))
      .map(([clave, valor]) => {

        // Se transforma el nombre del campo para que sea más legible para el usuario
        // Ejemplo: "nombre_colegio" → "Nombre Colegio"
        const etiqueta = clave
          // Reemplaza todos los guiones bajos "_" por espacios
          .replace(/_/g, ' ')

          // Busca la primera letra de cada palabra (\b = inicio de palabra)
          // y la convierte a mayúscula
          .replace(/\b\w/g, letra => letra.toUpperCase());

        return `
          <tr>
            <td>${etiqueta}</td>
            <td>${valor ?? '—'}</td>
          </tr>`;
      }).join('');

    // Se retorna una tabla HTML con los datos
    return `<table>${filas}</table>`;
  }
}

```

### En el componente: 
```javascript
  // Capa que define el icono que usarán los puntos (por ejemplo colegios)
  iconoColegio = {
    // Ruta de la imagen que se usará como icono del marcador
    iconUrl: 'leaflet/colegio.png',
    // Tamaño del icono en pixeles
    iconSize: [20, 20],
    // Punto del icono que se alineará con la coordenada del mapa
    iconAnchor: [17, 35],
    // Posición donde aparecerá el popup respecto al icono
    popupAnchor: [0, -35]
  };


  // Array donde se almacenarán todas las capas que se dibujen en el mapa
  // Cada objeto guarda el id de la capa, la capa Leaflet y si está visible
  capasGeoJSONalmacenadas: { id: string; capa: L.LayerGroup; visible: boolean }[] = [];


  // Método que se encarga de crear y registrar una capa en el mapa
  private gestionarCapa(id: string, datos: any[], opcionesPunto?: any, opcionesPoligono?: any): void {
    // Verificamos si la capa ya existe
    // Busca en el array si ya existe una capa con el mismo id
    const yaExiste = this.capasGeoJSONalmacenadas.some(c => c.id === id);
    // Si ya existe no se vuelve a dibujar
    if (yaExiste) {
      console.warn(`La capa "${id}" ya está dibujada, se omite`);
      return;
    }

    // Si no existe se crea la capa usando el servicio
    // El servicio dibuja las geometrías y retorna la capa y los límites
    const { capa, limites } = this.dibujarGeometria.dibujarGeometrias(datos, opcionesPunto, opcionesPoligono);
    // Se agrega la capa al mapa para que se muestre
    capa.addTo(this.mapa);
    // Se guarda la capa en el array para poder controlarla después
    this.capasGeoJSONalmacenadas.push({ id, capa, visible: true });
    // Se muestra en consola cuántas capas hay almacenadas
    console.log(`Capa "${id}" almacenada. Total: ${this.capasGeoJSONalmacenadas.length}`);
    // Si los límites son válidos se centra el mapa en la capa
    if (limites.isValid()) {
      this.mapa.fitBounds(limites, { padding: [40, 40] });
    }
  }


  // Método para eliminar una capa del mapa
  eliminarCapa(id: string): void {
    // Busca la posición de la capa dentro del array
    const indice = this.capasGeoJSONalmacenadas.findIndex(c => c.id === id);
    // Si no se encuentra la capa se muestra advertencia
    if (indice === -1) { console.warn(`Capa "${id}" no encontrada`); return; }
    // Se elimina la capa del mapa
    this.capasGeoJSONalmacenadas[indice].capa.remove();
    // Se elimina la capa del array almacenado
    this.capasGeoJSONalmacenadas.splice(indice, 1);
    // Mensaje informativo en consola
    console.log(`Capa "${id}" eliminada. Total: ${this.capasGeoJSONalmacenadas.length}`);
  }


  // Alterna la visibilidad de una capa sin eliminarla del array
  estadoVisibleCapa(item: { id: string; capa: L.LayerGroup; visible: boolean }): void {
    // Si la capa está visible
    if (item.visible) {
      // Se quita del mapa
      item.capa.remove();
      // Se marca como no visible
      item.visible = false;
    } else {
      // Si estaba oculta se vuelve a agregar al mapa
      item.capa.addTo(this.mapa);
      // Se marca como visible
      item.visible = true;
    }
  }


  // Método que obtiene y muestra los colegios
  mostrarColegios(): void {
    // Se llama al servicio que obtiene las escuelas desde el backend
    this.mapaServicio.listarEscuelas().pipe(
      tap((resp: any) => {
        // Si no hay datos se termina la ejecución
        if (!resp?.length) return;
        // Se crea la capa de colegios usando el icono definido
        this.gestionarCapa('colegios', resp, this.iconoColegio);
        // aqui puedo mandar null
      }),
      // Manejo de errores en la petición
      catchError(err => {
        console.error(err);
        return of([]);
      })
    ).subscribe();
  }


  // Método que obtiene y muestra los departamentos
  mostrarDepartamentos(): void {
    // Se llama al servicio que obtiene los departamentos
    this.mapaServicio.listarDepartamento().pipe(
      tap((resp: any) => {
        // Si no hay datos se detiene
        if (!resp?.length) return;
        // Se crea la capa de departamentos con estilo de polígono
        this.gestionarCapa('departamentos', resp, null, {
          color: '#1E40AF',       // color del borde
          colorRelleno: '#3B82F6',// color del relleno
          opacidadRelleno: 0.35,  // opacidad del relleno
          grosor: 2               // grosor del borde
        });
      }),
      // Manejo de errores de la petición
      catchError(err => {
        console.error(err);
        return of([]);
      })
    ).subscribe();
  }
```

### HTML
```html
  @if (activeTab === 'capas_almacenadas') {
    <div class="grid grid-cols-1 gap-2 p-2">

      @if (capasGeoJSONalmacenadas.length === 0) {
        <div class="text-center text-gray-500 text-sm p-3 border-round border-gray-200">
          No hay capas almacenadas
        </div>
      }

      @for (item of capasGeoJSONalmacenadas; track item.id) {

        <div class="flex items-center justify-between p-2 border rounded-2xl border-blue-500 border-round bg-white shadow-1">

          <!-- Nombre de la capa -->
          <span class="font-medium text-sm flex-1 truncate">
            {{ item.id }}
          </span>

          <!-- Botones -->
          <div class="flex gap-2">

            <p-button
              [icon]="item.visible ? 'pi pi-eye' : 'pi pi-eye-slash'"
              [severity]="item.visible ? 'success' : 'secondary'"
              size="small"
              rounded
              text
              pTooltip="{{ item.visible ? 'Ocultar capa' : 'Mostrar capa' }}"
              tooltipPosition="top"
              (click)="estadoVisibleCapa(item)">
            </p-button>

            <p-button
              icon="pi pi-trash"
              severity="danger"
              size="small"
              rounded
              text
              pTooltip="Eliminar capa"
              tooltipPosition="top"
              (click)="eliminarCapa(item.id)">
            </p-button>

          </div>

        </div>

      }

    </div>
  }

```

