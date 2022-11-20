# Práctica 3: Seguridad y calidad

Conseguir que el software funcione es solo la primera parte de crear un producto. Sin embargo, no es suficiente pensando el futuro. El software debe ser robusto, seguro y de calidad. La calidad se refiere a la capacidad de un producto de satisfacer las necesidades de los usuarios. La seguridad se refiere a la capacidad de un producto de protegerse de amenazas externas. La robustez se refiere a la capacidad de un producto de seguir funcionando en condiciones adversas.

En esta práctica vamos a ver cómo podemos asegurar la calidad de nuestro software para garantizar su funcionamiento en el futuro.

## Objetivos

1. Aplicar los conceptos de arquitectura hexagonal.
2. Implementar protección simple de endpoints.
3. Implementar autenticación JWT.

## Condiciones de entrega

1. La práctica se entrega en un repositorio de GitHub. Puede emplear el mismo respositorio de la práctica anterior.
2. Haga un commit por cada uno de los objetivos implementados.
3. Documente en markdown los resultados obtenidos y súbalo al respositorio donde ha realizado la práctica modificando el archivo `README.md`.

## Requisitos

Mismos requisitos de práctica 2.

## Recursos

* [Arquitectura Hexagonal](https://www.youtube.com/watch?v=VLhdDYaW-uI)
* [Para qué usar JWT](https://www.youtube.com/watch?v=3o4vEIkiRgE)
* [Qué es un token JWT](https://openwebinars.net/blog/que-es-json-web-token-y-como-funciona/)
* [Autenticación con NestJS](https://docs.nestjs.com/security/authentication)


## Desarrollo

### I. Arquitectura hexagonal

La arquitectura hexagonal es un patrón de diseño que nos permite separar la lógica de negocio de la lógica de infraestructura. De esta forma, podemos cambiar la infraestructura sin afectar a la lógica de negocio. Por ejemplo, podemos cambiar la base de datos sin afectar a la lógica de negocio.

Para esta parte de la práctica continúe con la entidad u objeto que decidió modelar para el proyecto de la práctica 2. Como ejemplo, vamos a usar la entidad `Player` que representa jugadores de fútbol.

1. Cree una rama llamada `hexagonal` a partir de `master` partiendo del código . Es posible hacerlo con el siguiente comando:
   
   ```bash
   git checkout -b hexagonal master
   ```

2. Modifique el código para que se aplique la arquitectura hexagonal. La estructura sera:
   
      - src
        - <nombre_de_entidad>
          - adapters
            - controllers
              - <nombre_entidad>.controller.ts
            - repositories
              - <nombre_entidad>.repository.ts (nuevo archivo)
          - domain
            - models
              - <nombre_entidad>.model.ts (nuevo archivo)
            - services
              - <nombre_entidad>.service.ts

   Para conseguir esto mueva los archivos originales `app.controller.ts` hacia la carpeta controllers. `app.service.ts`en services. Luego en VSCode seleccione en el explorador cada archivo y posteriormente en Rename para cambiar el nombre según la entidad que se haya definido en la práctica 2.

   Luego dentro de los archivos de controller y service modifique el nombre de la clase. Para hacerlo ubique el puntero sobre el nombre de la clase y presione F2, o con click secundario seleccione  `Rename Symbol`. Luego escriba el nuevo nombre y presione enter.

   Ejemplo de como usar [Rename Symbols en VSCode](https://www.youtube.com/watch?v=S1Jzrmw0ww8).

3. Para modelar los datos cree una carpeta `models` dentro de `domain` y cree un archivo de modelo. En cualquier aplicación existen registros de personas, es muy útil modelar una persona de manera abstracta y luego especificar los atributos. De esta manera se podrán agregar nuevos roles con sus especificaciones.

   Entonces el modelo de persona `person.model.ts`
   
   ```typescript
   export abstract class Person {
      name: string;
      lastName: string;
      age: number;
   }
   ```

   y ahora para extender el modelo a un jugador:

   ```typescript
   import { Person } from './person.model';

   export class Player extends Person {
      team: string;
   }
   ```

   Modelar los datos es muy útil para poder validarlos y para poder hacer consultas a la base de datos. Así mismo prevenir se ingresen datos a la fuerza en contextos donde no se deben, esto es algo que se debe cuidar particularmente en un lenguaje como JavaScript que es débilmente tipado.

4. Es momento de migrar la funcionalidad implementada en el controlador a un servicio. Para ello se emplea el archivo `player.service.ts` dentro de `services`, mueva la lógica del controlador a este servicio. Luego en el controlador importe el servicio y use sus métodos. Para el ejemplo planteado con jugadores el servicio será:

   ```typescript
   import { Injectable } from '@nestjs/common';
   // Importamos el modelo de jugador
   import { Player } from '../models/player.model';

   @Injectable()
   export class PlayerService {

      // Como no hay base de datos aun empleamos una variable en memoria:
      private player: Player[] = [{
         name: 'Leo',
         lastName: 'Messi',
         age: 35,
         team: 'Argentina'
      }]

      /**
       * Método para obtener todos los jugadores
       */
      public listar() : Player[] {
         return this.player
      }

      /**
       * Método para crear un jugador
       */
      public crear(jugador: Player): Player {
         this.player.push(jugador);
         return jugador;
      }

      /**
       * Método para modificar un jugador
       */
      public modificar(id: number, jugador: Player): Player {
            this.player[id] = jugador
            return this.player[id];
      }

      /**
       * Método para eliminar un jugador
       * Debido a que usamos un filtro, para validar si se elimina el jugador, 
       * primero se determina cuantos elementos hay en el arreglo y luego se hace una comparación.
       */
      public eliminar(id: number): boolean {
         const totalJugadoresAntes = this.player.length;
         this.player = this.player.filter((val, index) => index != id);
         if(totalJugadoresAntes == this.player.length){
            return false;
         }
         else{
            return true;
         }
      }

      /**
       * Método para modificar la edad de un jugador
       */
      public cambiarEdad(id: number, edad: number): Player {
         this.player[id].age = edad;
         return this.player[id];
      }

   }

   ```

5. Modificar el controlador `player.controller.ts` para que se emplee el la implementación del servicio. Adicionalmente se adicionan bloques try/catch para manejar los errores y han sido suprimidos los tipos de retorno de los métodos. El controlador quedará de la siguiente manera:

   ```typescript
   import { Body, Controller, Delete, Get, Param, Patch, Post, Put } from '@nestjs/common';
   import { PlayerService } from '../../domain/services/player.service';

   import {Player} from '../../domain/models/player.model';

   const errReturn = (e: Error, message: string) => {
      return {
         message: message,
         error: e
      }
   }

   @Controller()
   export class PlayerController {
   constructor(private readonly jugadorService: PlayerService) { }

      @Get()
      getHello() {
         try{
            return this.jugadorService.listar();
         }
         catch(e){
            return errReturn(e, "Error al listar jugadores");
         }
      }

      @Post()
      crear(@Body() datos: Player) {
         try{
            return this.jugadorService.crear(datos);
         }
         catch(e){
            return errReturn(e, "Error al crear jugador");
         }
      }

      @Put(":id")
      modificar(@Body() datos: Player, @Param('id') id: number) {
         try{
            return this.jugadorService.modificar(id, datos);
         }
         catch(e){
            return errReturn(e, "Error al modificar jugador");
         }
      }

      @Delete(":id")
      eliminar(@Param('id') id: number) {
         try{
            return this.jugadorService.eliminar(id);
         }
         catch(e){
            return errReturn(e, "Error al eliminar jugador");
         }
      }

      @Patch(":id/edad/:edad")
      cambiarEdad(@Param('id') id: number, @Param('edad') edad: number) {
         try{
            return this.jugadorService.cambiarEdad(id, edad);
         }
         catch(e){
            return errReturn(e, "Error al modificar edad del jugador");
         }
      }
   }

   ```

   [Siga este link para observar cómo quedó el código de ejemplo hasta este punto](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/tree/ec98cc61579fd73821b293e1deb389217a8a13c7).
 
6. (OPCIONAL) Es recomendable aplicar los principios SOLID para asegurar la mantebilidad y la economía de código a futuro.

   Los principios SOLID son:

   - **S**ingle Responsability Principle (Principio de Responsabilidad Única): Una clase debe tener una única responsabilidad y debe estar abierta a extensión pero cerrada a modificación.

   - **O**pen-Closed Principle (Principio de Abierto-Cerrado): Las entidades de software (clases, módulos, funciones, etc.) deben estar abiertas a la extensión pero cerradas a la modificación.

   - **L**iskov Substitution Principle (Principio de Sustitución de Liskov): Las entidades de software (clases, módulos, funciones, etc.) deben ser sustituibles por instancias de sus subtipos sin alterar la correctitud del programa.

   - **I**nterface Segregation Principle (Principio de Segregación de Interfaces): Las interfaces de software (clases, módulos, funciones, etc.) deben ser lo más pequeñas posibles.

   - **D**ependency Inversion Principle (Principio de Inversión de Dependencias): Las entidades de software (clases, módulos, funciones, etc.) deben depender de abstracciones y no de implementaciones.


   Hasta este punto se ha modelado una entidad u objeto, donde se han definido sus atributos y acciones, entonces se está aplicando el principio S e I. Ahora vamos a aplicar el principio D agregando interfaces para el controlador y el servicio. Esto tambien permitirá contener los comentarios de código separados de la implementación.

   [Siga este link para observar cómo quedó el código de ejemplo aplicando el principio D](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/commit/dc04de8b88572afe70bc9b334623a3ef10144b8b?diff=unified).

### II. Implementado seguridad 

Usualmente los servicios web requieren de seguridad para proteger la información que se envía y recibe. En este caso vamos a implementar un sistema de autenticación y autorización para que los usuarios puedan acceder a los recursos del servidor.

1. Instalar el paquete `@nestjs/passport` y `passport`, que permitirá implementar la autenticación y autorización.

   ```bash
   npm install --save @nestjs/passport passport passport-local
   npm install --save-dev @types/passport-local
   ```

2. NestJS integra un método para generar módulos rápidamente, primero se creará un módulo para autenticación con los siguientes comandos:

   ```bash
   nest g module auth
   nest g service auth
   ```

   Ahora aparecerá una carpeta de nombre `auth` en la carpeta `src` con los archivos `auth.module.ts` y `auth.service.ts` y un archivo `auth.service.spec.ts` que contiene las pruebas unitarias del servicio, este último no lo utilizaremos.

   Si la terminal responde con un error, es posible que no esté funcionando correctamente la configuración global del npm, para solucionarlo ejecute el siguiente comando:

   ```bash
   source ~/.profile
   ```

   Luego de eso, vuelva a ejecutar los comandos anteriores.

3. Asi mismo se creará un módulo para gestionar usuarios

   ```bash
   nest g module users
   nest g service users
   ```
4. Luego, se implementa el servicio de usuarios, para este ejemplo se utilizarán un par de usuarios predefinidos, pero en un caso real se debería implementar un servicio que permita gestionar usuarios.

   ```typescript
   import { Injectable } from '@nestjs/common';

   export type User = {
      userId: number,
      username: string,
      password: string
   };

   @Injectable()
   export class UsersService {
      private readonly users: User[] = [
         {
            userId: 1,
            username: 'john',
            password: 'changeme',
         },
         {
            userId: 2,
            username: 'maria',
            password: 'guess',
         },
      ];

      /**
         * Recupera los datos del usuario
         * @param username Nombre de usuario
         * @returns 
         */
      async findOne(username: string): Promise<User | undefined> {
         return this.users.find(user => user.username === username);
      }
   }
   ```

5. Ahora es necesario configurar el servicio para que esté disponible para otros servicios instanciados, en Nest se configuran como módulos, entonces se modifica el archivo `users.module.ts` así:

   ```typescript
   import { Module } from '@nestjs/common';
   import { UsersService } from './users.service';

   @Module({
      providers: [UsersService],
      exports: [UsersService], // Exporta el servicio
   })
   export class UsersModule {}
   ```

6. Es momento de implementar un servicio de autenticación, suele estar separado ya que el servicio de gestión de usuarios en una arquitectura de microservicios está aislada. Para obtener la funcionalidad esperada se modificará el archivo `auth.service.ts`

   ```typescript
   import { Injectable } from '@nestjs/common';
   import { UsersService } from '../users/users.service';

   @Injectable()
   export class AuthService {
   constructor(private usersService: UsersService) {}

      async validateUser(username: string, pass: string): Promise<any> {
         const user = await this.usersService.findOne(username);
         if (user && user.password === pass) {
            const { password, ...result } = user;
            return result;
         }
         return null;
      }
   }
   ```
   Ese servicio se encarga de validar que el usuario y la contraseña sean correctos, para eso se utiliza el servicio de usuarios.

7. Posteriormente hay que habilitar el servicio de gestión de usuarios en el módulo de autenticación, para eso se modifica el archivo `auth.module.ts`:

   ```typescript
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { UsersModule } from '../users/users.module';

   @Module({
      imports: [UsersModule], // Importa el módulo de usuarios
      providers: [AuthService]
   })
   export class AuthModule {}
   ```

8. Con lo anterior completado, el paso siguiente consiste en implementar una estrategia para validar al usuario, esto se consigue con el paquete `passport` que se instaló previamente. Esto se consigue creando un archivo con el nombre `local.strategy.ts` dentro de la carpeta `src/auth`.

   ```typescript
   import { Strategy } from 'passport-local';
   import { PassportStrategy } from '@nestjs/passport';
   import { Injectable, UnauthorizedException } from '@nestjs/common';
   import { AuthService } from './auth.service';

   @Injectable()
   export class LocalStrategy extends PassportStrategy(Strategy) {
      constructor(private authService: AuthService) {
         super();
      }

      async validate(username: string, password: string): Promise<any> {
         const user = await this.authService.validateUser(username, password);
         if (!user) {
            throw new UnauthorizedException();
         }
         return user;
      }
   }
   ```
 9.  Ahora se debe configurar el módulo de autenticación para que utilice la estrategia de autenticación, para eso se modifica el archivo `auth.module.ts`:

      ```typescript
      import { Module } from '@nestjs/common';
      import { AuthService } from './auth.service';
      import { UsersModule } from '../users/users.module';
      import { PassportModule } from '@nestjs/passport';
      import { LocalStrategy } from './local.strategy';

      @Module({
         imports: [UsersModule, PassportModule],
         providers: [AuthService, LocalStrategy]
      })
      export class AuthModule {}
      ```  

 10. Por último, es momento de proteger a los enpoints desarrollados previamente con el nuevo servicio, primero se modifica el controlador de la siguiente manera:

   ```typescript
   import { Body, Controller, Delete, Get, Inject, Param, Patch, Post, Put, UseGuards } from '@nestjs/common';
   ...
   import { AuthGuard } from '@nestjs/passport';

   ...

   @Controller()
   export class PlayerControllerImpl implements PlayerController {
   ...

   @UseGuards(AuthGuard('local')) // Se adiciona esta anotación
   @Post()
   create(@Body() datos: Player) ...
  ```
 11. Si todo está bien, en este punto al llamar al enpoint POST del servidor la respuesta debería ser esta:

    ```json
    {
      "statusCode": 401,
      "message": "Unauthorized"
    }
    ```

    Esto indica que está prohibido el acceso al recurso, ya que no se ha autenticado al usuario.

    Para conseguirlo puede emplearse el comando `curl` de la siguiente manera:

   ```bash
   curl -X POST http://localhost:3000 -d '{"username": "john", "password": "changeme", "name":"jugador"}' -H "Content-Type: application/json"
   ```

   En este punto debería crear un jugador, aunque tendrá un error, pues la contraseña estará contenida en el objeto que se guardará como un nuevo jugador:

   ```json
   {
      "name": "Cristiano Ronaldo",
      "lastname": "dos Santos Aveiro",
      "age": 37,
      "username": "john",
      "password": "changeme"
   }
   ```

   Además la contraseña es visible y sería vulnerable, por ello se utilizará una autenticación empleando JWT.

   [Puede observar el código de ejemplo hasta este punto en el este enlace](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/commit/2ec6c449b0d050dfe4ea65f02cd22b06056a7f19).

### III. Autenticación con JWT

1. Para implementar la autenticación con JWT se debe instalar el paquete `@nestjs/jwt`:

   ```bash
   npm install --save @nestjs/jwt passport-jwt
   npm install --save-dev @types/passport-jwt
   ```

2. Se modifica `auth.service.ts` adicionando el método login y algunas dependencias:

   ```typescript
   import { Injectable } from '@nestjs/common';
   import { UsersService } from '../users/users.service';
   import { JwtService } from '@nestjs/jwt';

   @Injectable()
   export class AuthService {
      constructor(
         private usersService: UsersService,
         private jwtService: JwtService
      ) {}

      async validateUser(username: string, pass: string): Promise<any> {
         const user = await this.usersService.findOne(username);
         if (user && user.password === pass) {
            const { password, ...result } = user;
            return result;
         }
         return null;
      }

      async login(user: any) {
         const payload = { username: user.username, sub: user.userId };
         return {
            access_token: this.jwtService.sign(payload),
         };
      }
   }
   ```

3. Para permitirle a los usuarios iniciar sesión se implementará un endpoint que convierta las credenciales de usuario en un token JWT. Para esto se crea un controlador `auth.controller.ts`:

   ```typescript
   import { Controller, Post, Request, UseGuards } from '@nestjs/common';
   import { AuthGuard } from '@nestjs/passport';
   import { AuthService } from './auth.service';

   @Controller('auth')
   export class AuthController {
      constructor(private authService: AuthService) {}

      @UseGuards(AuthGuard('local'))
      @Post('login')
      async login(@Request() req) {
         return this.authService.login(req.user);
      }
   }
   ```
4. Se crea un archivo de constantes donde se registra el secreto JWT, creando el archivo `constants.ts`:

   ```typescript
   export const jwtSecret = 'secretKey';
   ```
5. Posteriormente se adiciona una estrategia para permitirle Passport identificar donde se encontrará el token en una petición y cual es el secreto que permite validarlo. Se crea el archivo `jwt-auth.strategy.ts`:

   ```typescript
   import { ExtractJwt, Strategy } from 'passport-jwt';
   import { PassportStrategy } from '@nestjs/passport';
   import { Injectable } from '@nestjs/common';
   import { jwtSecret } from './constants';

   @Injectable()
   export class JwtStrategy extends PassportStrategy(Strategy) {
      constructor() {
         super({
            jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
            ignoreExpiration: false,
            secretOrKey: jwtSecret,
         });
      }

      async validate(payload: any) {
         return { userId: payload.sub, username: payload.username };
      }
   }
   ```


6. Ahora se modifica `auth.module.ts` para configurar correctamente el servicio de JWT

   ```typescript
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { LocalStrategy } from './local.strategy';
   import { UsersModule } from '../users/users.module';
   import { PassportModule } from '@nestjs/passport';
   import { JwtModule } from '@nestjs/jwt';
   import { AuthController } from './auth.controller';

   @Module({
      controllers: [AuthController],
      imports: [
         UsersModule,
         PassportModule,
         JwtModule.register({
            secret: "este es el secreto para generar JWT",
            signOptions: { expiresIn: '60m' },
         }),
      ],
      providers: [AuthService, LocalStrategy],
      exports: [AuthService],
      })
   export class AuthModule {}
   ```

8. Ahora es necesario implementar un guardia para que intercepte un token JWT y lo valide para proteger a los enpoints que sean de nuestro interés. Creamos en la carpeta `auth` un archivo llamado `jwt-auth.guard.ts`. 

   ```typescript
   import { Injectable, ExecutionContext, UnauthorizedException } from '@nestjs/common';
   import { AuthGuard } from '@nestjs/passport';

   @Injectable()
   export class JwtAuthGuard extends AuthGuard('jwt') {}
   ```   

9.  Se registran los nuevos componentes en el módulo `auth.module.ts`:

   ```typescript
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { LocalStrategy } from './local.strategy';
   import { UsersModule } from '../users/users.module';
   import { PassportModule } from '@nestjs/passport';
   import { JwtModule } from '@nestjs/jwt';
   import { AuthController } from './auth.controller';
   import { JwtAuthGuard } from './jwt-auth.guard';
   import { JwtStrategy } from './jwt.strategy';
   import { jwtSecret } from './constants';

   @Module({
      controllers: [AuthController],
      imports: [
         UsersModule,
         PassportModule,
         JwtModule.register({
            secret: jwtSecret,
            signOptions: { expiresIn: '60m' },
         }),
      ],
      providers: [AuthService, LocalStrategy, JwtStrategy, JwtAuthGuard],
      exports: [AuthService],
      })
   export class AuthModule {}
   ´´´

10. Si todo es correcto, será posible llamar al endpoint que genera un token JWT, esto se podrá validar con CURL con el siguiente comando:

   ```bash
   curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme" }' -H "Content-Type: application/json"
   ```

   La terminal responderá con un token. Guarde este token para usarlo en los siguientes pasos.

11. Proteger el endpoint POST 

   ```typescript
   import { Body, Controller, Delete, Get, Inject, Param, Patch, Post, Put, UseGuards } from '@nestjs/common';
   ...
   import { JwtAuthGuard } from 'src/auth/jwt-auth.guard';

   ...

   @Controller()
   export class PlayerControllerImpl implements PlayerController {
   ...

   @UseGuards(JwtAuthGuard) // Se adiciona esta anotación para proteger el endpoint
   @Post()
   create(@Body() datos: Player) ...
  ```

12. Si todo es correcto, será posible llamar al endpoint que genera un token JWT, esto se podrá validar con CURL con el siguiente comando, recuerde usar el token que generó en el paso 10:

    ```bash
    curl -X POST http://localhost:3000 -d '{"name": "Radamel", "lastname": "Falcao", "age": 36 }' -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2Vybm...""
    ```

    La terminal responderá con el objeto creado. Si no se envía el token o el token es incorrecto, la terminal responderá con un error 401.

    [Siga esta guía si sea usar Postman para validar](https://dev.to/loopdelicious/using-jwt-to-authenticate-and-authorize-requests-in-postman-3a5h)

    [Cambios hasta este punto](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/commit/66449212f3b63548776e48f00d8431f831a0d83b)

 13. Finalmente proteja los endpoints que tengan capacidad de modificar o eliminar registros, agrege la evidencia del funcionamiento de los endpoints protegidos con JWT al informe en el documento `README.md`.   

Resultado esperado:

   [Práctica 3 - Resultado de ejemplo](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/tree/Práctica-3)


## Licencia

Esta guía se redactado con la asistencia de GitHub Copilot 🤖.

Este material está bajo licencia [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

Universidad del Cauca - 2022.