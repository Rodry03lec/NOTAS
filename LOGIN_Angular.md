## TODO EL TRABAJO DE ANGULAR LOGIN

**Ya deberia estar descargado y todo**

### PASO I: ENVIRONMENTS

**Esto es para generar como la conexion unica conexion va ser a la api**

```
ng generate environments
```

**Ejemplo**

```
production: false,// maneja si es developer o produccion
apiUrl: 'http://127.0.0.1:8000/api' //la url
```

### PASO 2. SERVICIOS

**crear en donde este tus componentes**

```
ng generate service auth/servicios/auth --skip-tests
```

**Ejemplo**

```
import { inject, Injectable } from '@angular/core';
import { environment } from '../../../environments/environment';
import { HttpClient } from '@angular/common/http';


interface Credenciales{
  email: string;
  password: string;
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  constructor() { }

  //declarmaos la base url
  urlBase = environment.apiUrl;

  //inyectamos el http cliente
  http = inject(HttpClient);

  //metodo para ingresar al sistema
  login(credenciales:Credenciales){
    return this.http.post(`${this.urlBase}/v1/auth/login`, credenciales);
  }

  //para registrar datos
  registrar(datos:any){
    return this.http.post(`${this.urlBase}/v1/auth/registrar`, datos);
  }

  //para el ingreso al perfil
  perfil(){
    return this.http.get(`${this.urlBase}/v1/auth/perfil`);
  }


  //para cerrar la session
  logout(){
    return this.http.post(`${this.urlBase}/v1/auth/logout`, {});
  }
}
```

### PASO 3. GUARDS

#### en src/core/guards/auth
```
ng g guard core/guards/auth
```

**Servicios que controlan el acceso a las rutas de una aplicación Ejemplo:**

```
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';

export const authGuard: CanActivateFn = (route, state) => {

  //console.log("auth guard", route, state);
  //injectamos el router
  const router = inject(Router);
  //instanciamos el token
  const token = localStorage.getItem('access_token') || null;

  // Verifica si la ruta es el login
  const isLoginRoute = state.url.includes('/auth/login');

  if (!token && !isLoginRoute) {
    // Si no hay token y no está en la ruta de login, redirige al login
    router.navigate(['/auth/login']);
    return false;
  } else if (token && isLoginRoute) {
    // Si hay token y está en la ruta de login, a perfil
    router.navigate(['/admin/perfil']);
    return false;
  }

  return true;
};
```

### PASO 4. INTERCEPTORES

#### en src/core/interceptores/auth

```
ng g interceptor core/interfaces/auth
```

**Clases que permiten manipular las peticiones y respuestas HTTP de una aplicación**

```
import { HttpErrorResponse, HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { tap } from 'rxjs';

export const authInterceptor: HttpInterceptorFn = (req, next) => {

  //console.log('authInterceptor', req);
  //recupero el token
  const token = localStorage.getItem("access_token");
  //injectamos el router
  const router = inject(Router);

  //clono la peticion del requesst
  const peticion = req.clone({
    setHeaders:{
      Accept: 'aplication/json',
      Authorization: `Bearer ${token}`
    }
  });


  //retorno la peticion
  return next(peticion).pipe(tap(
    ()=>{},
    (error: any)=>{
      if(error instanceof HttpErrorResponse){
        if(error.status !== 401){
          return;
        }
        localStorage.removeItem("access_token");
        router.navigate(['/auth/login']);
      }
    }
  ));

  return next(req);
};
```

### PASO 5: HABILITAR LOS INTERCEPTORES

**Habilitar en app.config.ts principal**

```
//para importar
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './core/interceptores/auth.interceptor';

//habilitar los interceptores
provideHttpClient(withInterceptors([authInterceptor])),

```

### PASO 6: HABILITAR EN LAS RUTAS LOS GUARDS
**Ejemplo**
```
  {
    path: 'admin',
    component:AppLayout, //esto es para el menu
    children:[
      {
        path:'',
        loadChildren: ()=>import("./admin/admin.module").then(a=>a.AdminModule)
      }
    ],
    canActivate:[authGuard] ///aqui es donde se habilita
  }
```

### PASO 7: COMPONENTE DE LOGIN

**trabajamos con el COMPONENTE Login**
```
import { Component, inject } from '@angular/core';
import { AuthService } from '../servicios/auth.service';
import { Router } from '@angular/router';
import { FormControl, FormGroup } from '@angular/forms';

interface Credenciales{
  email: string;
  password: string;
}

@Component({
  selector: 'app-login',
  standalone: false,
  templateUrl: './login.component.html',
  styleUrl: './login.component.scss'
})
export class LoginComponent {
  //inyectamos el servicio
  authServicio = inject(AuthService);
  //iniciamos el rouer para la redireccion
  router = inject(Router);


  //Es un método del ciclo de vida de Angular que se ejecuta automáticamente después de que el componente se ha inicializado.
  ngOnInit(): void {
    // Obtiene el token del localStorage
    const token = localStorage.getItem('access_token') || null;

    // Verifica si el token existe
    if (token) {
      // Si el token existe, redirige al usuario al dashboard
      this.router.navigate(['/admin/perfil']);
    }
  }


  //para el login formulario
  loginFormulario = new FormGroup({
    email: new FormControl(''),
    password: new FormControl('')
  });

  //para ingresar el funcion login
  funLogin(){
    this.authServicio.login(this.loginFormulario.value as Credenciales).subscribe(
      (respuesta:any)=>{
        //console.log(respuesta);
        //guardar el token en localStorage
        localStorage.setItem("access_token", respuesta.access_token);
        //redireccionamos al administrador
        this.router.navigate(['/admin/perfil']);
      },
      (error)=>{
        console.log(error);
        alert("error al autenticar");
      }
    );
  }
}
```
**Trabajamos con la vista**
```
<div [formGroup]="loginFormulario" >
    <label for="email" class="block text-surface-900 dark:text-surface-0 text-xl font-medium mb-2">Email</label>
    <input pInputText id="email" formControlName="email" type="text" placeholder="Ingrese su usuario" class="w-full md:w-[30rem] mb-8" autocomplete="off"/>

    <label for="password" class="block text-surface-900 dark:text-surface-0 font-medium text-xl mb-2">Password</label>
    <p-password id="password" formControlName="password"  placeholder="Ingrese su contraseña" [toggleMask]="true" styleClass="mb-4" [fluid]="true" [feedback]="false"></p-password>

    <div class="py-4">
    <p-button label="Ingresar" styleClass="w-full" (click)="funLogin()"></p-button>
    </div>

</div>
```

### PASO 8: MOSTRAMOS EL PERFIL
**Para la parte de perfil componente**
```
import { Component, inject } from '@angular/core';
import { AuthService } from '../../../auth/servicios/auth.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-perfil',
  standalone: false,
  templateUrl: './perfil.component.html',
  styleUrl: './perfil.component.scss'
})
export class PerfilComponent {
  perfil:any = {};
  //importamos el auth servicio que esta creaod en auth/servicios/auth
  authServicio = inject(AuthService);
  //el router mas
  router = inject(Router);

  constructor(){
    this.funPerfil();
  }

  //para el perfil
  funPerfil(){
    this.authServicio.perfil().subscribe(
      (resp)=>{
        this.perfil = resp;
      },
      (error)=>{
        alert("error al recuperar el perfil"+error)
      }
    );
  }

  //para salir y cerrar  session
  funSalir(){
    this.authServicio.logout().subscribe(
      ()=>{
        localStorage.removeItem("access_token");
        this.router.navigate(['/auth/login']);
      },
      (error)=>{
        alert("Error al salir: "+ error);
      }
    );
  }
}
```

**Para la vista**
```
<div class="card">
  <h2>NOMBRE: {{ perfil.name }}</h2>
  <h2>EMAIL: {{ perfil.email }}</h2>
  <p-button label="Cerrar Sesion" [raised]="true" severity="danger" (click)="funSalir()" />
</div>
```
