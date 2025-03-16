## PASOS PARA LA IMPORTACION DE SAKAI EN ANGULAR
### Descargar Angular con vite o normal
```
npm create vite@latest
```

### Descargar de github el template 
```
https://github.com/primefaces/sakai-ng
```
## IMPORTAR EL TEMPLATE 
## **1. Configurar el package.json. debes observar dependencias y devdependencias**

#### DEPENDENCIAS 
```
"@primeng/themes": "^19.0.5",
"chart.js": "4.4.2",
"primeclt": "^0.1.5",
"primeicons": "^7.0.0",
"primeng": "^19.0.5",
"tailwindcss-primeui": "^0.3.2",
```
#### DEVDEPENDENCIAS
```
"autoprefixer": "^10.4.20",
"postcss": "^8.4.49",
"tailwindcss": "^3.4.17",
```
### INSTALAR LOS PAUQETES
```
npm install
```
## **2. Copiar tailwind.config.js y llevar al proyecto**
## **3. Copiar lo que tenga de src/styles.scss**
```
@use './tailwind.css';
@use './assets/layout/layout.scss';
@use 'primeicons/primeicons.css';
// @use './assets/demo/demo.scss';
```
## **4. Copiar tailwind.css a src/**
## **5. Copiar assets a mi proyecto a src/**
## **6. En src/app/app.config.ts**
```
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import Aura from '@primeng/themes/aura';
import { providePrimeNG } from 'primeng/config';
import { provideHttpClient } from '@angular/common/http';

//importamos
provideAnimationsAsync(),
providePrimeNG({ theme: { preset: Aura, options: { darkModeSelector: '.app-dark' } } })
```
## **7. En el index.html**
```
<link href="https://fonts.cdnfonts.com/css/lato" rel="stylesheet" />
```
## **8. Copiar de app/layout en app/**

## **9. Habilitar el template en app.routes.ts**
```
{
  path:'admin',
  component: AppLayout, //este se habilita a todo el modulo
  children:[
    {
      path: '',
      loadChildren: ()=>import('./admin/admin.module').then(m=>m.AdminModule)
    }
  ]
},
```

