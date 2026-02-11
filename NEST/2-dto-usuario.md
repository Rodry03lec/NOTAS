# DTO USUARIO
### dto : *Data Transfer Object*

## Instalar pg para Conexion a Postgres
```
npm install pg
```
## Configurar
### En database/database.module.ts
#### cuantas conexiones tendra depende
```
import { Global, Module } from "@nestjs/common";

import { Pool } from 'pg';

@Global()
@Module({
    providers:[
        {
            provide: 'PG_POOL_C1',
            useFactory: ()=>{
                return new Pool({
                    host: '10.1.102.117',
                    port: 5432,
                    user: 'usuario',
                    password: 'password',
                    database: 'gisdb'
                })
            }
        },
        {
            provide: 'PG_POOL_C2',
            useFactory: ()=>{
                return new Pool({
                    host: '10.1.102.117',
                    port: 5432,
                    user: 'usuario',
                    password: 'password',
                    database: 'gisdb'
                })
            }
        }
    ],
    exports: ['PG_POOL_C1', 'PG_POOL_C2']
})
export class DatabaseModule{}
```

#### Importa en app.module.ts
```
  imports: [
    DatabaseModule,
  ]
```

### Instalaciones
```
npm install @nestjs/mapped-types
// Instalar
npm i bcrypt
npm i -D @types/bcrypt
import * as bcrypt from 'bcrypt';

```

## modules/usuarios

### dto/create-usuario.dto.ts
```
import { IsEmail, IsNotEmpty, IsString, Matches, MaxLength, MinLength } from "class-validator";

// instalar :: npm i --save class-validator class-transformer
export class CreateUsuarioDto{

    @IsString()
    @IsNotEmpty({ message: 'El nombre es obligatorio' })
    @MinLength(2, { message: 'El nombre debe tener al menos 2 caracteres' })
    @MaxLength(50, { message: 'El nombre no debe exceder 50 caracteres' })
    nombre: string;

    @IsString()
    @IsNotEmpty({ message: 'El usuario es obligatorio' })
    @MinLength(2)
    @MaxLength(50)
    usuario:string;

    @IsEmail({}, { message: 'Debe ingresar un correo valido'})
    @IsNotEmpty({ message: 'El correo debe ser obligatorio' })
    correo:string;

    @MinLength(8, { message: 'La contraseña debe tener al menos 8 caracteres ' })
    @MaxLength(64, { message: 'La contraseña es demasiado larga' })
    @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/, { message: 'La contraseña debe contener al menos una mayúscula, una minúscula y un número',})
    password: string;
}
```

### dto/update-usuario.dto.ts
```
// npm install @nestjs/mapped-types
import { PartialType } from '@nestjs/mapped-types';
import { CreateUsuarioDto } from "./create-usuario.dto";

export class UpdateUsuarioDto extends PartialType(CreateUsuarioDto) {}
```

### usuario.controller.ts
```
import { Body, Controller, Delete, Get, HttpCode, HttpStatus, Param, ParseIntPipe, Patch, Post } from "@nestjs/common";
import { UsuarioService } from "./usuario.service";
import { CreateUsuarioDto } from "./dto/create-usuario.dto";
import { UpdateUsuarioDto } from "./dto/update-usuario.dto";

@Controller("usuario")
export class UsuarioController{

    constructor( private usuarioServicio: UsuarioService ){}

    // Para listar los usuarios
    @Get("listarUsuario")
    funListarUsuario(){
        return this.usuarioServicio.usuarioListar();
    }

    // Para crear nuevo usuario
    @Post("nuevoUsuario")
    @HttpCode(HttpStatus.CREATED)
    funNuevoUsuario(@Body() nuevoUsuario:CreateUsuarioDto){
        const usuario = this.usuarioServicio.nuevoUsuario(nuevoUsuario);

        return usuario;
    }

    // Para ver un usuario
    @Post("verUsuario")
    funVerUsuario(@Body() dato:any){
        const usuario = this.usuarioServicio.verUsuario(dato.id);
        return usuario;
    }

    // Para eliminar un usuario
    @Delete("eliminarUsuario")
    funEliminarUsuario( @Body() datos: any ){
        return this.usuarioServicio.eliminarUsuario(datos.id);
    }


    @Patch('updateUsuario/:id')
    actualizar( @Param('id', ParseIntPipe) id: number, @Body() dto: UpdateUsuarioDto ) {
        return this.usuarioServicio.guardarUsuario(id, dto);
    }

}
```

### usuario.service.ts
```
import { Pool } from 'pg';
import { BadRequestException, Inject, Injectable, InternalServerErrorException, NotFoundException } from "@nestjs/common";
import { CreateUsuarioDto } from './dto/create-usuario.dto';

import * as bcrypt from 'bcrypt';
import { UpdateUsuarioDto } from './dto/update-usuario.dto';

@Injectable()
export class UsuarioService{

    constructor(
        @Inject('PG_POOL_C1') private readonly conexionUno: Pool
    ){}

    // Para listar usuarios
    async usuarioListar(){
        try {
            const resultado = await this.conexionUno.query(`
                select id, nombre , correo, usuario from public.usuario;
            `);

            return {
                tipo: 'success',
                resultado: resultado.rows
            };
        } catch (error) {
            console.error('Error en usuarioListar:', error);
            throw new InternalServerErrorException('Error al obtener los usuarios');
        }
    }

    // Para crear un nuevo usuario
    // Instalar
    // npm i bcrypt
    // npm i -D @types/bcrypt
    // import * as bcrypt from 'bcrypt';
    async nuevoUsuario(crearUsuarioDto:CreateUsuarioDto){

        const hashedPassword = await bcrypt.hash(crearUsuarioDto.password, 12);
        const correo = crearUsuarioDto.correo.trim().toLowerCase();

        
        // para la transaccion del cliente pool
        const transaccionCliente = await this.conexionUno.connect();

        try {
            // Iniciamos la transaccion
            await transaccionCliente.query('BEGIN');

            // Ejecutamos la insersion de usuario
            const resultado = await transaccionCliente.query(`
                INSERT INTO usuario (nombre, usuario, correo, password)
                VALUES ($1, $2, $3, $4)
                RETURNING id, nombre, usuario, correo
            `, [
                crearUsuarioDto.nombre,
                crearUsuarioDto.usuario.trim().toLowerCase(),
                correo,
                hashedPassword
            ]);

            const  nuevoUsuario = resultado.rows;
            
            // si todo esta bien confirmamos 
            await transaccionCliente.query('COMMIT');

            return nuevoUsuario;

        } catch (error) {
            
            await transaccionCliente.query('ROLLBACK');

            if (error.code === '23505') {
                if (error.constraint === 'correo_unique') {
                throw new BadRequestException({
                    field: 'correo',
                    message: 'El correo ya está registrado',
                });
                }

                if (error.constraint === 'usuario_unique') {
                throw new BadRequestException({
                    field: 'usuario',
                    message: 'El usuario ya está en uso',
                });
                }
            }

            throw new InternalServerErrorException('Error al procesar el registro');

        } finally{
            transaccionCliente.release();
        }
    }

    // para capturar datos de un solo usuario
    async verUsuario(id:number){
        try {
            const resultado = await this.conexionUno.query(`
                select id, nombre , correo, usuario 
                from public.usuario
                where id = $1 
            `, [id]);
            return resultado.rows;
        } catch (error) {
            throw new InternalServerErrorException('Error al procesar el registro');
        }
    }

    // Para eliminar un usuario
    async eliminarUsuario(id:number){
        try {
                const resultado = await this.conexionUno.query(`
                    DELETE FROM public.usuario
                    WHERE id = $1
                    RETURNING id
                `, [id] );

                if (resultado.rowCount === 0) {
                    throw new NotFoundException('Usuario no encontrado');
                }

                return {
                    tipo: 'success',
                    resultado: `Usuario con id ${id} eliminado`
                };
        } catch (error) {
            throw new InternalServerErrorException('Error al procesar el registro');
        }
    }

    // guardar usuarioeditado
    async guardarUsuario(id: number, editarDto: UpdateUsuarioDto) {
        try {

            const resultado = await this.conexionUno.query(`
                UPDATE public.usuario
                SET nombre = $1,
                    usuario = $2,
                    correo = $3
                WHERE id = $4
                RETURNING id, nombre, usuario, correo
            `,[
                editarDto.nombre?.trim(),
                editarDto.usuario?.trim().toLowerCase(),
                editarDto.correo?.trim().toLowerCase(),
                id 
            ]);

            if (resultado.rowCount === 0) {
                throw new NotFoundException('Usuario no encontrado');
            }

            return {
                tipo: 'success',
                resultado: resultado.rows[0]
            };

        } catch (error: any) {
            if (error.code === '23505') {
                throw new BadRequestException('Usuario o correo ya existe');
            }
            if (error instanceof NotFoundException) {
                throw error;
            }
            throw new InternalServerErrorException('Error al actualizar usuario');
        }
    }

}
```
### usuario.module.ts
```
import { Module } from "@nestjs/common";
import { UsuarioController } from "./usuario.controller";
import { UsuarioService } from "./usuario.service";

@Module({
    imports:[],
    controllers:[UsuarioController],
    providers:[UsuarioService]
})
export class UsuarioModule{

}
```

