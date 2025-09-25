## PASOS PARA LA IMPORTACION DE SAKAI EN ANGULAR 20
### Descargar Angular con vite o normal
#### ojo esto es para la ultima version
```
npm create vite@latest 
```
## Puedes descargar la version con npx
```
npx -p @angular/cli@20 ng new nombre_proyecto
```

### Descargar de github el template 
```
https://github.com/primefaces/sakai-ng
```
## IMPORTAR EL TEMPLATE 
## **1. Configurar el package.json. debes observar dependencias y devdependencias**

#### DEPENDENCIAS 
```
"@angular/animations": "^20",


"@primeuix/themes": "^1.2.1",
"chart.js": "4.4.2",
"primeclt": "^0.1.5",
"primeicons": "^7.0.0",
"primeng": "^20",
"rxjs": "~7.8.2",
"tailwindcss-primeui": "^0.6.1",
"tslib": "^2.8.1",
"zone.js": "~0.15.1"

```
#### DEVDEPENDENCIAS
```
"autoprefixer": "^10.4.20",

"postcss": "^8.5.6",
"prettier": "^3.6.2",
"tailwindcss": "^4.1.11",
```
### INSTALAR LOS PAUQETES
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
@use './assets/layout/layout.scss';
@use 'primeicons/primeicons.css';
//@use './demo/demo.scss';
```
### tailwind.css
```
@import 'tailwindcss';
@import 'tailwindcss-primeui';
@custom-variant dark (&:where(.app-dark, .app-dark *));

@theme {
    --breakpoint-sm: 576px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 992px;
    --breakpoint-xl: 1200px;
    --breakpoint-2xl: 1920px;
}


```
## **4. Copiar assets a mi proyecto a src/app/ sin tailwind  y styles**

## **5. En src/app/app.config.ts**
```
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import Aura from '@primeuix/themes/aura';
import { providePrimeNG } from 'primeng/config';
import { provideHttpClient } from '@angular/common/http';

//importamos
provideAnimationsAsync(),
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
      loadChildren: ()=>import('./admin/admin.module').then(m=>m.AdminModule)
    }
  ]
},
```