# Práctica 1: Configuración de entorno y red

Aunque Internet hace parte de la vida diaria, no todos comprenden cómo funciona la comunicación por medio de esta red, la comprensión del protocolo IP, su importancia en el contexto de comunicación de cualquier servicio, la diferencia entre protocolos TCP y UDP la conveniencia según el tipo de servicio es útil para la diseño e implementación de servicios en la IoT.

Es de particular interés la comprensión de que es el protocolo TCP/IP que es la base de internet.

Debido a las necesidades de esta y las siguientes prácticas, para configurar un ambiente funcional en linux donde utilizan herramientas como Github y Docker, que implican la comprensión de algunos conceptos básicos se considera también como un componente de la práctica. 

## Objetivos de la práctica

Con el desarrollo de la práctica se espera que el estudiante aprenda como:

1. Crear un entorno Linux virtual para evaluar servicios empleando herramientas como VirtualBox, Github y Docker.
2. Identificar en Linux mecanismos para la identificación de ocupación de puertos y a qué servicio está relacionado
3. Implementar un pequeño servicio cliente/servidor en Python empleando TCP y UDP, a partir de ello establezca cual es más adecuado para algunos contextos que le serán planteados.

## Requisitos

1. Requerimientos de base
2. Python 3.8 o superior

## Desarrollo de la práctica

1. Configuración del entorno:  El estudiante deberá configurar su dispositivo de elección para ejecutar una imagen virtualizada de Linux, este será un suministro importante para el resto de prácticas.
2. Instalar docker
3. Reconocimiento de herramientas de red: Identificar configuración de red por medio del comando ip e ifconfig. Identificar servicios y puertos ocupados en el sistema con los comandos ss, netstat y lsof. 
4. Identificar servicios desplegados: El estudiante deberá identificar 5 servicios diferentes listados por las herramientas de red y determinar a qué aplicaciones posiblemente están relacionados.
5. Evaluar scripts en Python: Al estudiante se le entregarán scripts en Python para desplegar un ejemplo de cliente servidor con protocolos TCP y UDP, el estudiante evaluará el rendimiento de los dos servicios y debe descubrir la ocupación de los puertos por medio de las herramientas previamente estudiadas.


## Evaluación

El estudiante deberá documentar sus resultados en un pequeño documento en Markdown. El archivo será publicado por medio de un repositorio en Github donde registarán todos los resultados de esta y las siguientes prácticas.

## Recursos

### Conceptos
* [Qué es el modelo TCP/IP (Internet)](https://www.youtube.com/watch?v=1pB2kan_AFk)
* [Protocolo UDP](https://www.youtube.com/watch?v=4MSwQ9aIzq0)
* [Video explicativo sobre Docker](https://www.youtube.com/watch?v=9eTVZwMZJsA) (recomendado)
* [Comando ss](https://phoenixnap.com/kb/ss-command)
* [Comando nettstat](https://geekflare.com/es/netstat/)
* [Comando lsof](https://geekflare.com/es/lsof-command-examples/)
* [Servidor Python TCP](https://rico-schmidt.name/pymotw-3/socket/tcp.html)
* [Servidor Python UDP](https://rico-schmidt.name/pymotw-3/socket/udp.html)
* [Sintaxis Markdown](https://markdown.es/sintaxis-markdown/)
### Guías de instalación

* [Instalación de Docker en Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
* [Instalación de Docker en Debian](https://aprendiendoavirtualizar.com/instalar-docker-en-debian-11/)
* [Instalación de Docker en Raspbian](https://www.simplilearn.com/tutorials/docker-tutorial/raspberry-pi-docker)
* [Actualizar Python en Rasbian](https://raspberrytips.com/install-latest-python-raspberry-pi/)
* [Conectar VSCode con SSH a RPi](https://cloudbytes.dev/snippets/develop-remotely-on-raspberry-pi-using-vscode-remote-ssh) (opcional)
* [Markdown en VSCode](https://www.youtube.com/watch?v=pTCROLZLhDM)


# Desarrollo

Sigue el desarrollo de la práctica con el siguiente [video](https://drive.google.com/file/d/1yl5uZHS83F5UVLdD0pUzIRKmmETlocDk/view?usp=share_link)

# Licencia

Este material está bajo licencia [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

Universidad del Cauca - 2022.