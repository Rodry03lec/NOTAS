## PASOS PARA LA IMPORTACION DE SAKAI EN ANGULAR 21
### Descargar Angular

## Verificar:
## **1. Configurar el package.json. debes observar dependencias y devdependencias**

### DEPENDENCIAS 
```
"@angular/platform-browser-dynamic": "^21",    // El motor de arranque: Permite que Angular se compile y ejecute en el navegador en tiempo real (JIT).

"@primeuix/themes": "^2.0.0",     // El motor de diseño: Contiene los nuevos "Design Tokens" (colores, espacios) que alimentan el look de PrimeNG.
"@tailwindcss/postcss": "^4.1.11",  // El procesador: El plugin oficial para que Tailwind v4 funcione dentro de tu flujo de trabajo de PostCSS.
"chart.js": "4.4.2",  // Gráficos: Librería externa necesaria para que los componentes de gráficas de PrimeNG (p-chart) funcionen.
"primeicons": "^7.0.0",  // Iconod: La librería oficial de iconos vectoriales para usar en botones, menús y inputs.
"primeng": "^21.0.2",  // Librería de UI: El conjunto principal de componentes (tablas, formularios, modales) listos para usar.

"tailwindcss-primeui": "^0.6.1",  // El puente: Plugin que conecta las variables de PrimeUI con las clases de Tailwind CSS.
```
### DEVDEPENDENCIAS
```
"autoprefixer": "^10.4.20",

"postcss": "^8.5.6",
"prettier": "^3.0.0",
"tailwindcss": "^4.1.11",
```

## INSTALAR LOS PAUQETES
```
npm install
```

## **2. Copiar .postcssrc.json y llevar al proyecto**
```
{
    "plugins": {
      "@tailwindcss/postcss": {}
    }
}
```
## **3. Copiar lo que tenga de assets/styles.scss y tailwind.css**
## **A src/**
### styles.scss 
```
/* You can add global styles to this file, and also import other style files */
@use './tailwind.css';
@use 'primeicons/primeicons.css';
@use './assets/layout/layout.scss';
//@use '@/assets/demo/demo.scss';
```
### tailwind.css
```
@import 'tailwindcss';

@plugin 'tailwindcss-primeui';

@custom-variant dark (&:where([class*="app-dark"], [class*="app-dark"] *));

@theme {
  /* --breakpoint-*: initial; */
  --breakpoint-sm: 576px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 992px;
  --breakpoint-xl: 1200px;
  --breakpoint-2xl: 1920px;
}

/*
  The default border color has changed to `currentcolor` in Tailwind CSS v4,
  so we've added these compatibility styles to make sure everything still
  looks the same as it did with Tailwind CSS v3.

  If we ever want to remove these styles, we need to add an explicit border
  color utility to any element that depends on these defaults.
*/
@layer base {
  *,
  ::after,
  ::before,
  ::backdrop,
  ::file-selector-button {
    border-color: var(--color-gray-200, currentcolor);
  }
}

```

## **4. Copiar _assets_ a mi proyecto a src/app/ sin tailwind  y styles**

## **5. En src/app/app.config.ts**
```
import { provideHttpClient, withFetch } from '@angular/common/http';


import Aura from '@primeuix/themes/aura';
import { providePrimeNG } from 'primeng/config';

provideHttpClient(withFetch()),
    providePrimeNG({ theme: { preset: Aura, options: { darkModeSelector: '.app-dark' } } })
```

## **6. En el index.html**
```
<link href="https://fonts.cdnfonts.com/css/lato" rel="stylesheet" />
```

## **7. Copiar de app/layout en app/**

## **8. Habilitar el template en app.routes.ts**
```
{
  path:'admin',
  component: AppLayout, //este se habilita a todo el modulo
  children:[
    {
      path: '',
      loadChildren: () => import('./web/admin/admin.routes').then(m => m.adminRoutes),
    }
  ]
},
```

## **9. para crear la ruta ordenada**
```
import { Route } from '@angular/router';
import { Admin } from './components/admin/admin';

export const adminRoutes: Route[]=[
  {
    title: 'IDG - ADMIN',
    path:'',
    component:Admin
  }
]

```
