# Práctica 4: Desplegando en la nube

Desarrollar un servidor web en la nube es una tarea relativamente sencilla. En esta práctica vamos a desplegar una aplicación web en la nube utilizando un servicio de hosting gratuito.

## Objetivos
1. Desplegar una aplicación en la nube.
2. Agregar una base de datos a la aplicación.

## Condiciones de entrega

1. En moodle, entregar un enlace a la aplicación desplegada en la nube.
2. En el repositorio de la práctica, incluir un archivo `README.md` con la documentación de los endpoints de la API para su validación.
   
## Requisitos

Mismos requisitos de práctica 2.

## Recursos

* [TypeORM y NestJS](https://www.youtube.com/watch?v=AeaQRY8xFPk)
* [Deploy NodeJs app con Deta](https://www.youtube.com/watch?v=AcDrVnpd4I8)
* Heroku fúe una herramienta gratuita, aún siendo de pago es muy útil. Si está interesado [siga esta guía](https://www.youtube.com/watch?v=glDv0baCYzU).
* [Desplegar app NestJS con AWS Fargate](https://www.youtube.com/watch?v=ra_kJFIpU4A)

## Desarrollo

### 1. Desplegar una aplicación en la nube

Inicialmente esta práctica se realizó utilizando el servicio de hosting gratuito de [Heroku](https://www.heroku.com/). Sin embargo, este servicio ha cambiado su política de uso y ahora requiere una tarjeta de crédito para poder utilizarlo. Por ello, se ha optado por utilizar el servicio de hosting gratuito de [Deta](https://www.deta.sh/).

1. Descargar deta ejecutando en la terminal:
      
         curl -fsSL https://get.deta.dev/cli.sh | sh

2. Agregando la variable de entorno:

         echo "export PATH=~/.deta/bin:$PATH" >> ~/.bashrc

         source ~/.bashrc

3. Iniciar sesión en Deta ejecutando en la terminal:

         deta login

   Se ejecutará su navegador por defecto y se le pedirá que inicie sesión en su cuenta de Deta. Si no tiene una cuenta, puede crear una [aquí](https://web.deta.sh/signup).

4. Crear un punto de entrada de la aplicación. El archivo `src/index.ts` debe quedar de la siguiente forma:

   ```typescript
   import { NestFactory } from '@nestjs/core';
   import { ExpressAdapter } from '@nestjs/platform-express';
   import { AppModule } from './app.module';

   const createNestServer = async (expressInstance) => {
   const app = await NestFactory.create(
      AppModule,
      new ExpressAdapter(expressInstance),
   );

   return app.init();
   };

   export default createNestServer;
   ```

5. En la raíz del proyecto crear un nuevo archivo index.js con el siguiente contenido:

   ```javascript
   const express = require('express');
   const createServer = require('./dist/index').default;

   const app = express();
   let nest;

   app.use(async (req, res) => {
   if (!nest) {
      nest = express();
      await createServer(nest);
   }
   return nest(req, res);
   });

   module.exports = app;
   ```

6. Antes de publicar debemos compilar el proyecto. Para ello ejecutamos en la terminal:

         cd ~/Documents/Servidores/practica_02
         nest build

7. Publicar la aplicación ejecutando en la terminal:

         
         deta new --node 
         

   Si todo es correcto en la terminal tendrá la siguiente salida:
   
      ```bash
      Successfully created a new micro
      {
            "name": "UC_Practicas_IoT_Servidor",
            "id": "43b3e1f0-2720-44ee-b34f-949f1b9ec478",
            "project": "b0y5eo5t",
            "runtime": "nodejs14.x",
            "endpoint": "https://vgu9jp.deta.dev",
            "region": "us-east-1",
            "visor": "disabled",
            "http_auth": "disabled"
      }
      Adding dependencies...

      > @nestjs/core@9.2.0 postinstall /tmp/node_modules/@nestjs/core
      > opencollective || exit 0

                                 Thanks for installing nest 
                     Please consider donating to our open collective
                              to help us maintain this package.
                                             
                                 Number of contributors: 0
                                    Number of backers: 864
                                 Annual budget: $180,908
                                 Current balance: $4,176
                                             
                  Become a partner: https://opencollective.com/nest/donate
                                             

      + passport-jwt@4.0.0
      + reflect-metadata@0.1.13
      + @nestjs/jwt@9.0.0
      + passport-local@1.0.0
      + passport@0.6.0
      + rxjs@7.5.7
      + rimraf@3.0.2
      + @nestjs/core@9.2.0
      + @nestjs/passport@9.0.0
      + @nestjs/common@9.2.0
      + @nestjs/platform-express@9.2.0
      added 133 packages from 142 contributors and audited 133 packages in 19.617s

      15 packages are looking for funding
      run `npm fund` for details

      found 0 vulnerabilities
      ```

8. Para actualizar la aplicación ejecutamos en la terminal:

         deta update

9. Para desplegar la aplicación ejecutamos en la terminal:

         deta deploy <nombre_proyecto>

10. Para activar los logs de la aplicación ejecutamos en la terminal:

         deta visor enable

11. Para obtener la url del servicio visite el panel de control de Deta en la sección de micros. `https://web.deta.sh/micros`. Ahí encontrará en micros una opción con el nombre del proyecto, al abrirlo encontrará la url del servicio.

    Para el ejemplo se obtuvo la url `https://d35sk9.deta.dev/`, al visitarla debería obtener la respuesta implementada en el método get de su servicio, por ejemplo:

    ```json
      [
         {
            "name": "Leo",
            "lastName": "Messi",
            "age": 35,
            "team": "Argentina"
         }
      ]
    ```

### 2. Conectado a una base de datos

Para conectar una base de datos emplearemos la librería TypeORM. Este es un ORM que permite trabajar con bases de datos relacionales y no relacionales. En este caso emplearemos una base de datos no relacional orientada a documentos llamada MongoDB, muy popular por su flexibilidad y alto rendimiento.

1. Crear una cuenta gratuita en MongoDB Atlas. [Aquí](https://www.mongodb.com/cloud/atlas) puede crear una cuenta gratuita. 
   
2. Presione el botón `Build a Cluster` y seleccione la opción `Free Tier`, se recomienda elegir AWS como proveedor de infraestructura y presione el botón `Create Cluster`.

3. Creado el cluster, es necesario crear un usuario, para este ejemplo emplee Username and password. En los campos correspondientes ingrese un nombre de usuario y una contraseña. Presione el botón `Add User`.

4. Para facilitar la conexión la base de datos se recomienda crear una IP whitelist, para ello presione el botón `Add IP Address` y seleccione la opción `Allow Access from Anywhere`. Presione el botón `Confirm`. Si no encuentra esta opción ingrese IP Address y en el campo asigne el valor `0.0.0.0`, finalmente presione el botón `Add Entry`.

5. Guarde y despliegue haciendo click en el botón `Finish and close`. En este punto ya tendrá una base de datos MongoDB lista para ser utilizada.
   
6. Instalar las dependencias de TypeORM y MongoDB ejecutando en la terminal:

         npm install --save @nestjs/typeorm typeorm mongodb

7. En el panel de mongo atlas, en la sección `Connect` seleccione la opción `Connect your application`. Copie la url de conexión y reemplace el valor de la variable `MONGO_URL` en el archivo `~/Documents/Servidores/practica_02/src/app.module.ts` por la url de conexión.

8. Modifique el archivo `src/app.module.ts` para que quede de la siguiente manera:

   ```typescript
   import { TypeOrmModule } from '@nestjs/typeorm';
   ...

   @Module({
   imports: [
      AuthModule,
      UsersModule,
      TypeOrmModule.forRoot({
         type: 'mongodb',
         url: 'mongodb+srv://<usuario>:<password>...',
      }),
   ],
   controllers: [PlayerControllerImpl],
   providers: [
      {
         provide: 'PlayerService',
         useClass: PlayerServiceImpl,
      },
   ],
   })
   export class AppModule {}
   ```

9. Cree un archivo para modelar la entidad, eso queda en la carpeta `src/<nombre_entidad>/domain/entities` y debe llamarse `<nombre_entidad>.entity.ts`, en este caso `player.entity.ts`. El archivo debe quedar de la siguiente manera:

   ```typescript
   import { Entity, Column, ObjectIdColumn } from 'typeorm';

   @Entity()
   export class PlayerEntity {
      @ObjectIdColumn()
      id: string;

      @Column()
      name: string;

      @Column()
      lastName: string;

      @Column()
      age: number;

      @Column()
      team: string;
   }
   ```

10. Agrege la entidad en la configuración del módulo, para habilitar el repositorio:

   ```typescript
  
   @Module({
   imports: [
      ...
      TypeOrmModule.forRoot({
         type: 'mongodb',
         url: 'mongodb+srv://<usuario>:<password>...',
         useNewUrlParser: true,
         useUnifiedTopology: true,
         synchronize: true, // Solo para desarrollo
         logging: true,
         autoLoadEntities: true,
      }),
      TypeOrmModule.forFeature([PlayerEntity])
   ],
   ...
   export class AppModule {}
   ```

11. Luego adicione en el constructor del servicio, para el ejemplo `src/player/domain/services/player.service.ts`:

   ```typescript
   ...
   @Injectable()
   export class PlayerServiceImpl implements PlayerService {
   constructor(
      @InjectRepository(PlayerEntity)
      private repository: MongoRepository<PlayerEntity>,
   ) {}
   ...
   ```

12. Idealmente en esta etapa se implementan puertos (ports), que se encargan a adaptar los datos de aplicación (models) a los datos del repositorio (entities). Para simplificar el ejemplo se omitirá esta etapa. Entonces se reemplazará la implementación del modelo `Player` por la entidad `PlayerEntity` en el servicio `PlayerServiceImpl`. De igual manera, como los datos toman tiempo para ser capturados del repositorio, se emplean Promesas. El cambio de `Player` a `PlayerEntity` se aplica en los servicios y controladores.

12. Ahora es necesario modificar los métodos para que utilicen el repositorio, para el ejemplo `src/player/domain/services/player.service.ts`:

   ```typescript
   ...
   public async list(): Promise<PlayerEntity[]> {
      return await this.repository.find();
   }

   public async create(playerData: PlayerEntity): Promise<InsertResult> {
      const newPlayer = await this.repository.insert(playerData);
      return newPlayer;
   }

   public async update(
      id: number,
      playerData: PlayerEntity,
   ): Promise<UpdateResult> {
      const updatedPlayer = await this.repository.update(id, playerData);
      return updatedPlayer;
   }

   public async delete(id: number): Promise<boolean> {
      const deleteResult = await this.repository.delete(id);
      return deleteResult.affected > 0;
   }

   public async updateAge(id: number, edad: number): Promise<UpdateResult> {
      const updatedPlayer = await this.repository.update(id, { age: edad });
      return updatedPlayer;
   }
   ...
   ```

13. Finalmente, si se ejecuta el proyecto localmente con `npm run start:dev`, ejecute un POST para crear un nuevo elemento. Si la respuesta es satisfactoria y si al ejecutar el GET se recupera el elemento creado, entonces la conexión con la base de datos fue exitosa.

14. Actualice el proyecto en la nube:

      ```bash
      git add .
      git commit -m "Conexión con MongoDB"
      git push heroku master

15. Esta implementación no funcionará con DETA, debido a restricciones de la capa gratuita de la plataforma. Para poder utilizar la base de datos, se debe adquirir una cuenta de pago, alternativamente puede solicitar créditos en AWS para desplegar sin restricciones.

## Licencia

Esta guía se redactado con la asistencia de GitHub Copilot 🤖.

Este material está bajo licencia [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

Universidad del Cauca - 2022.