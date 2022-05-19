#### Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc)

Subir todo lo elaborado a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación  y un archivo README.md con instrucciones detalladas para compilar y desplegar la aplicación, tanto en una PC local como en la nube (AWS o GCP).


## Solución

Para ejecutar el archivo docker-compose.yml, utilizaremos el comando: 

``` docker-compose up --build```

Con este comando haremos el build del docker'compose. En el caso de **django, nginx**, se llama a los archivos dockerfile que se encuentran en las carpetas backend y frontend. El dockerfile es una archivo que contiene los comandos para generar una imagen de docker, en los mismos se encuentran la imagen de docker de donde se parte (en varias ocasiones se usó alpine, por ser una imagen liviana). En el caso de **postgresql**, no se uso un archivo dockfile, sino que se declaró en el mismo compose, haciendo uso de una imagen postgres-alpine y seteando las variables de entorno en .envs/.postgres

El docker-compose es fundamental, ya que nos permite enviar ciertos parametros, por ejemplo los mapeos de los puertos a usar, los paths de los archivos, y si un servicio depende de otro, para que espere hasta que se termine de crear el primero antes de ejecutar el segundo servicio

Se decidió elegir Postgresql, ya que es muy comun usarlo en conjunto a django y nginx (Además que uno de los requerimientos era la librería psycopg2 de python que es justamente para utilizar Postgresql). Podremos observar la aplicacion corriendo en **http://localhost:8000/** 

Importante: En el caso de utilizar **AWS**, hay que crear una inbound rule para el security group que use la instancia que usemos para habilitar el puerto que estemos usando para la aplicacion, en este caso el 8000

![Diagrama AWS](aws-securitygroup.png)

### Deployear con Kubernetes

Si queremos deployear con kubernetes, lo que tendremos que hacer es traducir el docker compose a algo que entiende kubernetes:

En el directorio del docker-compose.yml ejecuto el comando kompose convert, pero esto nos creara muchos archivos y lo que queremos hacer es dirigirlo a un solo archivo, para eso usamos la opcion -o de output. El comando quedaria:

``` kompose convert -o ./kompose-output```

Luego utilizamos kubectl para implementar el docker-compose convertido con el kompose.

``` kubectl apply -f ./kompose-output```
