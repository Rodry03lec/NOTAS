# Habilitar servicios REST en Moodle

## 1. Habilitar Web Services

### Site administration > Advanced features
- Activar: **Enable web services**
- Guardar cambios

---

## 2. Habilitar protocolo REST

### Site administration > Server > Web services > Manage protocols
- Activar: **REST protocol**
- Guardar cambios

---

## 3. Crear servicio externo

### Site administration > Server > Web services > External services > Add
- **Name:** miServicio
- **Short name:** (opcional)
- Marcar: **Enabled**
- Guardar cambios

---

## 4. Agregar funciones al servicio

Dentro del servicio creado:

- Agregar función:
  - `core_course_get_courses`

> Puedes agregar más funciones según lo que necesites.

---

## 5. Generar token

### Site administration > Server > Web services > Manage tokens > Create token

- **User:** seleccionar usuario (recomendado: administrador)
- **Service:** miServicio ---> el servicio que se creo en el paso 3
- **Valid until:** opcional (puedes dejarlo sin fecha)
- Crear token

---

## 6. Usar el token

Ejemplo de endpoint:


Parámetros básicos:

- `wstoken=TU_TOKEN`
- `wsfunction=core_course_get_courses`
- `moodlewsrestformat=json`



# instalaciones necesarias para nest
```bash
    npm install @nestjs/axios
```

## Te crear un modulo especifo y ahi lo importas
```typescript
import { HttpModule } from '@nestjs/axios';

@Module({
  imports: [HttpModule],
})
export class MoodleModule {}
```

## En el servicio de igual manera
```typescript
import { Injectable } from '@nestjs/common';

import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class MoodleService {

    private baseUrl = 'http://localhost:8080/webservice/rest/server.php';
    private token = 'a69cea60b4eff2f05a43ef996d4b6f82';

    constructor(private http: HttpService) {}


    // Para listar un curso
    async listarCursos() {
        const params = new URLSearchParams();

        params.append('wstoken', this.token);
        params.append('wsfunction', 'core_course_get_courses');
        params.append('moodlewsrestformat', 'json');

        const response = await firstValueFrom(
            this.http.post(this.baseUrl, params)
        );

        return response.data;
    }


    //

    private async llamarMoodle(params: URLSearchParams) {
        const response = await firstValueFrom(
            this.http.post(this.baseUrl, params)
        );
        return response.data;
    }

    // para crear categoria
    async crearCategoria(name: string) {
        const params = new URLSearchParams();
        params.append('wstoken', this.token);
        params.append('wsfunction', 'core_course_create_categories');
        params.append('moodlewsrestformat', 'json');
        params.append('categories[0][name]', name);
        params.append('categories[0][parent]', '0');

        return await this.llamarMoodle(params);
    }

    // listar categorias


    // crear un curso
    async crearCurso(fullname: string, shortname: string, categoryid: number) {
        const params = new URLSearchParams();
        params.append('wstoken', this.token);
        params.append('wsfunction', 'core_course_create_courses');
        params.append('moodlewsrestformat', 'json');
        params.append('courses[0][fullname]', fullname);
        params.append('courses[0][shortname]', shortname);
        params.append('courses[0][categoryid]', categoryid.toString());

        return await this.llamarMoodle(params);
    }


    // Para crear usuuario
    async crearUsuario(user: any) {
        const params = new URLSearchParams();
        params.append('wstoken', this.token);
        params.append('wsfunction', 'core_user_create_users');
        params.append('moodlewsrestformat', 'json');

        params.append('users[0][username]', user.username);
        params.append('users[0][password]', user.password);
        params.append('users[0][firstname]', user.firstname);
        params.append('users[0][lastname]', user.lastname);
        params.append('users[0][email]', user.email);
        params.append('users[0][idnumber]', user.idnumber);

        return await this.llamarMoodle(params);
    }

    // 

    async inscribirUsuario(userid: number, courseid: number) {
        const params = new URLSearchParams();
        params.append('wstoken', this.token);
        params.append('wsfunction', 'enrol_manual_enrol_users');
        params.append('moodlewsrestformat', 'json');

        params.append('enrolments[0][roleid]', '5');
        params.append('enrolments[0][userid]', userid.toString());
        params.append('enrolments[0][courseid]', courseid.toString());

        return await this.llamarMoodle(params);
    }

    // duplicar un curso
    async duplicarCursoCompleto(data: any) {
        const params = new URLSearchParams();

        params.append('wstoken', this.token);
        params.append('wsfunction', 'core_course_duplicate_course');
        params.append('moodlewsrestformat', 'json');

        params.append('courseid', data.courseid.toString());
        params.append('fullname', data.fullname);
        params.append('shortname', data.shortname);
        params.append('categoryid', data.categoryid.toString());
        params.append('visible', '1');

        const response = await this.llamarMoodle(params);

        return response;
    }

}
```

## El controlador
```typescript
import { Body, Controller, Get, Post } from '@nestjs/common';
import { MoodleService } from './moodle.service';

@Controller('moodle')
export class MoodleController {

    constructor(private moodleServicio: MoodleService) {}


    // http://localhost:3000/api/moodle/listar-cursos
    @Get('listar-cursos')
    funListarCursos() {
        return this.moodleServicio.listarCursos();
    }

    /*
        http://localhost:3000/api/moodle/crear-categoria
        {
            "name": "Programación"
        }
    */
    // Para crear categoria
    @Post('crear-categoria')
    funCrearCategoria(@Body() body: any) {
        return this.moodleServicio.crearCategoria(body.name);
    }

    /*
        http://localhost:3000/api/moodle/crear-curso
        {
            "fullname": "Curso NEST",
            "shortname": "NODE2026",
            "categoryid": 2
        }
    */
    // Para crear un curso
    @Post('crear-curso')
    funCrearCurso(@Body() body: any) {
        return this.moodleServicio.crearCurso(
            body.fullname,
            body.shortname,
            body.categoryid
        );
    }

    /*
    http://localhost:3000/api/moodle/crear-usuario
    {
        "username": "usuario2",
        "password": "User123*",
        "firstname": "Adrian",
        "lastname": "Lecoña Quispe",
        "email": "adrian@gmail.com",
        "idnumber": "12345678"
    }
    */
    // Para crear un usuario
    @Post('crear-usuario')
    funCrearUsuario(@Body() body: any) {
        return this.moodleServicio.crearUsuario(body);
    }

    /*
    http://localhost:3000/api/moodle/agregar-usuario-curso
    {
        "userid": 5,
        "courseid": 4
    }
    */
    @Post('agregar-usuario-curso')
    enrolUser(@Body() body: any) {
        return this.moodleServicio.inscribirUsuario(
            body.userid,
            body.courseid
        );
    }

    /*
    http://localhost:3000/api/moodle/duplicar-curso
    {
        "courseid": 4,
        "fullname": "Copia Curso NEST",
        "shortname": "COPIA DE NEST 2026",
        "categoryid": 2
    }
    */
    // para duplciar curso
    @Post('duplicar-curso')
    funDuplicarCurso(@Body() body: any) {
        return this.moodleServicio.duplicarCursoCompleto(body);
    }
}
```

## solo como ejemplo ojo