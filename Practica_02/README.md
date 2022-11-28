# Práctica 2: Creando un servidor REST

En la [práctica 1](https://github.com/mfdogalindo/UC-Practicas-SWoT/tree/main/Practica_01) se exploraron los recursos necesarios para el entorno donde se va desplegar el servidor, hasta ese punto se está cumpliendo con la implementación de internet, donde se cuenta con un recurso de red que expone un puerto TCP/IP para la comunicación con el cliente. 

En esta práctica se completará la implementación de un servidor Web, es decir, empleando [tecnologías de la World Wide Web](https://www.w3.org/standards/webarch/) expondremos información con el protocolo de [Transferencia de Estado Representacional](https://es.wikipedia.org/wiki/Transferencia_de_Estado_Representacional) conocido como REST.

## Objetivos

1. Instalar servidor NodeJS y NestJS.
2. Implementar un servidor web que exponga un recurso REST.
3. Publicar en GitHub el código fuente del servidor.
4. Identificar los verbos Http y su uso para un caso de ejemplo.

## Condiciones de entrega

1. La práctica se entrega en un repositorio de GitHub.
2. Haga un commit por cada uno de los objetivos implementados.
3. Documente en markdown los resultados obtenidos y súbalo al respositorio donde ha realizado la práctica modificando el archivo `README.md`.
## Requisitos

1. Requisitos de base.
2. NodeJS, se recomienda una versión LTS.

## Recursos

### Conceptos

* [Los verbos Http y su uso en REST](https://codigofacilito.com/articulos/verbos-http-rest)
* [Curso base de Javascript](https://youtu.be/ivdTnPl1ND0)
* [Guía de Javascript de Mozilla](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide)
* [Fundamentos de typescript](https://www.coursera.org/projects/fundamentos-ts)
* [Bases de NestJS](https://www.youtube.com/watch?v=NYoCbihISxw)
* [Documentación NestJS](https://docs.nestjs.com/)

### Conceptos opcionales

* [Arquitectura Hexagonal](https://medium.com/@edusalguero/arquitectura-hexagonal-59834bb44b7f)
* [Clean Code](https://openwebinars.net/cursos/clean-code/)


### Desarrollo

#### I. Instalar NodeJS y NestJS

1. En debian, ubuntu, raspbian, etc. Abrir la terminal y ejecutar el comando:
   
         sudo apt update && sudo apt install nodejs -y


   En Arch con paru:

         paru -S nodejs 

2. Actualizar Nodejs:

         sudo npm cache clean -f
         sudo npm install -g n
         sudo n stable

   A veces con debian las versiones disponibles de NodeJs no incluyen el gestor de paquetes `npm`. Si el comando npm no está disponible, instalarlo con:

         sudo apt install npm

   Después de esto ejecutar nuevamente los comandos de actualización

3. Validar las versiones instaladas, ejecute:

         node -v

   La terminal responderá con la última versión LTS de node disponible

         npm -v

   La terminal responderá con la última versión de npm disponible.

4. Creando un espacio para los recursos globales en nodejs:

         cd
         mkdir ~/.npm-global
         npm config set prefix '~/.npm-global'
         echo "export PATH=~/.npm-global/bin:$PATH" >> ~/.bashrc
         source ~/.bashrc

   Nota: Si emplea zsh como shell, en lugar de ~/.profile, emplee ~/.zshrc

5. Finalmente, para instalar NestJS se deben ejecutar los comandos:

         npm i -g @nestjs/cli
         source ~/.bashrc

#### II. Ejecutando el ejemplo Hello World

NestJS al tratarse de un framework facilita la creación rápida de un servicio.

1. Para iniciar seleccione o cree una carpeta donde se alojará el proyecto, por ejemplo:

         cd ~/Documents
         mkdir Servidores
         cd Servidores

2. Crear un proyecto NestJS ejecutando el comando nest new, para el nombre del proyecto no usar espacios ni caracteres especiales:

         nest new <nombre-proyecto>

   Ejemplo:
   
         nest new practica_02

La terminal preguntará si se desea usar yarn o npm, se recomienda usar npm.

      ? Which package manager would you like to use? (Use arrow keys)
      ❯ npm 
        yarn

3. Es necesario identificar la dirección IP de la máquina, para esto se puede ejecutar el comando:

         hostname -I

   En la terminal se mostrará la dirección IP de la máquina, por ejemplo:

         192.168.128.13 172.17.0.1

4. Node utiliza el archivo `package.json` para definir los scripts que se ejecutan con el comando `npm run` o *yarn run*. Para ejecutar el ejemplo Hello World, se debe montar la carpeta y ejecutar el comando:

         cd practica_02
         npm run start:dev

   Para verificar los scripts disponibles, se puede ejecutar el comando:

         cat package.json

   La terminal mostrará el contenido del archivo `package.json`. Que es una estructura de datos en formato JSON que contiene información sobre el proyecto, como el nombre, la versión, los autores, las dependencias, los scripts, etc.

   ```json
   {
   "name": "practica_02",
   "version": "0.0.1",
   "description": "",
   "author": "",
   "private": true,
   "license": "UNLICENSED",
   "scripts": {
      "prebuild": "rimraf dist",
      "build": "nest build",
      "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
      "start": "nest start",
      "start:dev": "nest start --watch",
      "start:debug": "nest start --debug --watch",
      "start:prod": "node dist/main",
      "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
      "test": "jest",
      "test:watch": "jest --watch",
      "test:cov": "jest --coverage",
      "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
      "test:e2e": "jest --config ./test/jest-e2e.json"
   },
   ...
   ```

   Para detener la ejecución del servidor, presione `Ctrl + C`.

   Si la ejecución es correcta se observará la siguiente información en la terminal:

         [Nest] 1   - 2021-03-01 21:00:00   [NestFactory] Starting Nest application...
         [Nest] 1   - 2021-03-01 21:00:00   [InstanceLoader] AppModule dependencies initialized +0ms
         [Nest] 1   - 2021-03-01 21:00:00   [RoutesResolver] AppController {/}: +1ms
         [Nest] 1   - 2021-03-01 21:00:00   [RouterExplorer] Mapped {/, GET} route +1ms
         [Nest] 1   - 2021-03-01 21:00:00   [NestApplication] Nest application successfully started +1ms

5. Con lo anterior el servidor nos indica que está listo para recibir peticiones con el metodo GET en la ruta raíz. Por defecto el servidor escucha en el puerto `3000`, para verificar esto se puede ejecutar el comando en otra terminal:

         netstat -tulpn | grep node

   La terminal responderá con la siguiente información:

         (Not all processes could be identified, non-owned process info will not be shown, you would have to be root to see it all.)
         tcp6       0      0 :::3000                 :::*                    LISTEN      9604/node

   Luego ya se puede probar el servidor con el comando en otra terminal:

         curl http://localhost:3000

   La terminal responderá con la siguiente información:

         Hello World!

   Así mismo en un navegador web en la máquina host se puede ingresar a la dirección `http://{{*direccion IP de la máquina virtual*}}:3000` y se mostrará la misma información.

         http://192.168.128.13:3000

   Dependiendo de la configuración de la máquina host, el servidor puede estar expuesto en toda la red local, permitiendo que otros dispositivos en la misma red puedan acceder al recurso. 


#### III. Publicando el código en GitHub

1. Para publicar el código en GitHub, se debe crear una cuenta en la página web de GitHub, si ya tiene una cuenta puede omitir este paso.
2. Crear un repositorio en GitHub, para esto se debe ingresar a la página web de GitHub y seleccionar el botón `New` en la sección `Repositories`.

3. En la página de creación de repositorio, se debe ingresar el nombre del repositorio, en este caso se usará el mismo nombre del proyecto, luego seleccionar la opción `Public` y finalmente seleccionar el botón `Create repository`.
4. En la máquina virtual, se debe ingresar a la carpeta del proyecto y ejecutar los siguientes comandos:

   Montar carpeta del proyecto:

         cd ~/Documents/Servidores/practica_02

   Configure la llave SSH para evitar ingresar la contraseña cada vez que se ejecuta el comando. Para configurar la autenticación SSH siga [este tutorial](https://medium.com/humantodev/configurar-ssh-github-enwindows-10-linux-y-macos-e843eb6d104e).

   Luego, inicializar repositorio con el url SSH del repositorio creado en GitHub:

         git remote add origin <url_del_repositorio>

   Por ejemplo:      
         
         git remote add origin git@github.com:mfdogalindo/UC_Practicas_IoT_Servidor.git

   Configurar usuario de Git:

         git config --global user.name "mfdogalindo"
         git config --global user.email "mfdogalindo@gmail.com"
      

         git init
         git add .
         git commit -m "Primer commit"
         git push --set-upstream origin master
            
   Finalmente al ejecutar el comando para sincronizar con GitHub:

         git push --set-upstream origin master

   Se debe observar la siguiente información en la terminal:

         Enumerating objects: 20, done.
         Counting objects: 100% (20/20), done.
         Delta compression using up to 4 threads
         Compressing objects: 100% (20/20), done.
         Writing objects: 100% (20/20), 148.30 KiB | 1.95 MiB/s, done.
         Total 20 (delta 0), reused 0 (delta 0), pack-reused 0
         To github.com:mfdogalindo/UC_Practicas_IoT_Servidor.git
         * [new branch]      master -> master
         Branch 'master' set up to track remote branch 'master' from 'origin'.

   Para verificar que el código se ha publicado correctamente, se puede ingresar a la página web de GitHub y verificar que el repositorio se ha creado correctamente.

#### IV. Los verbos HTTP

Para facilitar el desarrollo de la práctica se recomienda emplear Remote SSH en Visual Studio Code, para esto se debe instalar la extensión Remote - SSH en Visual Studio Code. Para instalar la extensión se debe seleccionar el botón *Extensions* en la sección *View* y luego buscar la extensión *Remote - SSH* y seleccionar el botón *Install*.

   **Nota**: *Para que funcione VSCode Remote es necesario instalar algunos recursos en la máquina virtual, tambien es indispensable tener SSH instalado y configurado. Para ciertos dispositivos como la Raspberry Pi 1 y Zero 1 no hay compatibilidad con VSCode Remote. En ese caso es útil WinSCP para transferir archivos entre la máquina virtual y la máquina host.*

1. Conecte VSCode a la máquina virtual, para esto se debe seleccionar el botón *Remote-SSH: Connect to Host...* en la sección *Remote Explorer*.
2. En el botón *Open Folder* seleccione la carpeta del proyecto. 

         /home/<User>/Documents/Servidores/practica_02/
3. Ingrese contraseña del usuario de la máquina virtual.
4. Si ha conectado correctamente, en el explorador de VSCode, seleccione el archivo `app.controller.ts` que se encuentra en la carpeta `src`.
5. Dentro del archivo encontrará una función precedida de la anotación @Get()

   ```typescript
     @Get()
      getHello(): string {
         return this.appService.getHello();
      }
   ```

   Esta funcion será ejecutada cada vez que se haga una petición al servidor, por defecto un navegador Web ejecuta una petición Get, por esta razón el método anterior se ejecuta cuando se ingresa la dirección IP y puerto del servidor en el navegador Web.

   Para cambiar el comportamiento del servidor, se debe modificar el método anterior, por ejemplo, para que el servidor responda con un mensaje de texto, se debe modificar el método anterior de la siguiente manera:

   ```typescript
      @Get()
      getHello(): string {
         return "Hola mundo";
      }
   ```

   Luego de hacer los cambios, se debe guardar el archivo y ejecutar el comando en caso de no estar ejecutando el servidor:

         npm run start:dev

   Para verificar que el servidor responde con el mensaje de texto, se debe ingresar la dirección IP del servidor en el navegador Web o usando el comando CURL como se hizo previamente.

   ```bash
      curl http://<ip_del_servidor>:3000
   ```

6. Creando un modificador
   
   Primero se creará una variable para guardar un mensaje adicional, y esta variable será retornada por el método getHello().

      ```typescript
      private persona = "Mundo";

      @Get()
      getHello(): string {
         return `Hola: ${this.persona}`
      }
      ```

   Después se creará un método para modificar el mensaje adicional, y este método será invocado como un POST utilizando la ruta como entrada del parámetro `nombre`, entonces `app.controller.ts` quedará de la siguiente manera:

      ```typescript
      import { Controller, Get, Param, Post } from '@nestjs/common';
      import { AppService } from './app.service';

      @Controller()
      export class AppController {
         constructor(private readonly appService: AppService) {}

         private persona = "Mundo";

         @Get()
         getHello(): string {
            return `Hola: ${this.persona}`
         }

         @Post(':nombre')
         modificar(@Param('nombre') nombre: string): string {
            this.persona = nombre;
            return `Mensaje modificado: ${this.persona}`
         }
      }
      ```

   Al guardar, el servidor automáticamente se reiniciará y se podrá probar el nuevo método usando CURL o el navegador Web.

      ```bash
      curl -X POST http://<ip_del_servidor>:3000/Pikachu
      ```

   La salida será:

      ```bash
      Mensaje modificado: Pikachu
      ```

   Recuerde que el método POST no se podrá llamar desde el navegador.

   Para validar el cambio en la variable desde el navegador, se debe llamará nuevamente al servidor al método GET como se ha hecho anteriormente. En el navegador se observará el mensaje:

      ```bash
      Hola: Pikachu
      ```

7. Suba los cambios realizados a GitHub invocando en la terminal de la maquina virtual los siguientes comandos:

      ```bash
      cd ~Documents/Servidores/practica_02
      git add .
      git commit -m "Se agrego un metodo POST"
      git push origin master
      ```


8. Seleccione un tema que considere que es posible modelar como una entidad en base de datos, por ejemplo personas, animales, productos, etc. Experimente con las anotaciones @Put(), @Delete() y @Patch(). Recuerde que estas anotaciones deben ser importadas desde el paquete @nestjs/common. Aunque usualmente VSCode le asistirá con la importación, si no lo hace, puede hacerlo manualmente.
   
   Con las otras anotaciones se espera que el estudiante pueda comprender el funcionamiento de los verbos HTTP. Recordando los siguientes criterios:

   * @Post() - Se usa para crear un recurso.
   * @Put() - Actualiza un recurso.
   * @Delete() - Elimina un recurso.
   * @Patch() - Actualiza un recurso parcialmente.

   Entonces, para cada una de las anotaciones utilice los verbos para crear, modificar, eliminar y actualizar, es decir un CRUD para modificar una entidad. Si tiene dudas de cómo hacerlo siga este [ejemplo sobre personas](https://github.com/mfdogalindo/UC_Practicas_IoT_Servidor/blob/Ejemplo-Práctica-2/src/app.controller.ts).


   Si desea profundizar más en el tema, puede consultar la documentación oficial de NestJS en el siguiente enlace: https://docs.nestjs.com/controllers

   Tambien se recomienda esta [guía para el uso de parámetros en rutas](https://desarrolloweb.com/articulos/parametros-rutas-nest).

   Opcionalmente puede emplear la recepción de parámetros por medio de Body para los verbos POST, PUT y PATCH. Para esto se debe importar la anotación @Body() desde el paquete @nestjs/common. Esta [guía le puede resultar útil](https://desarrolloweb.com/articulos/atender-solicitudes-post-nestjs).

   Como herramienta para evaluar el funcionamiento de los verbos  emplear [Postman](https://www.youtube.com/watch?v=qsejysrhJiU) o [Insomnia](https://www.youtube.com/watch?v=DkK4jOw7YLU) le resultará útil. 

9.  En el archivo `README.md` del repositorio, agregue la siguiente información:

       * Nombre completo
       * Nombre de la asignatura
       * Fecha de realización

      Explicación de los pasos realizados en el numeral 8 (Implementación de los verbos HTTP).

10. Haga commit y push a GitHub invocando en la terminal de la maquina virtual los siguientes comandos:

      ```bash
      cd ~Documents/Servidores/practica_02
      git add .
      git commit -m "Se completó la tarea"
      git push origin master
      ```
   

# Licencia

Esta guía se redactado con la asistencia de GitHub Copilot 🤖.

Este material está bajo licencia [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

Universidad del Cauca - 2022.