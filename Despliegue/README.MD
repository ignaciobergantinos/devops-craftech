#### Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc)

Subir todo lo elaborado a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación  y un archivo README.md con instrucciones detalladas para compilar y desplegar la aplicación, tanto en una PC local como en la nube (AWS o GCP).


## Solucion

Para ejecutar el archivo docker-compose.yml, utilizaremos el comando: 

``` docker-compose up --build```

Con este comando haremos el build del docker'compose. En el caso de **django, nginx**, se llama a los archivos dockerfile que se encuentran en las carpetas backend y frontend. El dockerfile es una archivo que contiene los comandos para generar una imagen de docker, en los mismos se encuentran la mangen de docker de donde se parte (en varias ocasiones se usó alpine, por ser liviano). En el caso de **postgresql**, no se uso un archivo dockfile, sino que se declaro en el mismo compose, haciendo uso de una imagen postgres-alpine y declarando las variables de entorno en un archivo llamado .postgres.

El docker-compose es fundamental, ya que nos permite enviar ciertos parametros, por ejemplo los mapeos de los puertos a usar, los paths de los archivos, y si un servicio depende de otro, para que espere hasta que se termine de crear el primero antes de ejecutar el nuevos servicio

Se decidió elegir Postgresql, ya que es muy comun usarlo en conjunto a django y nginx (Además que uno de los requerimientos era la libreria psycopg2 de python que es justamente para utilizar Postgresql). Podremos observar la aplicacion corriendo en **http://localhost:8000/** 

Importante: En el caso de utilizar **AWS**, hay que crear una inbound rule para el security group que use la instancia que usemos para habilitar el puerto que estemos usando para la aplicacion, en este caso el 8000

![Diagrama AWS](aws-securitygroup.png)

### Deployear con Kubernetes

Si queremos deployear con kubernetes, lo que tendremos que hacer es traducir el docker compose a algo que entiende kubernetes:

En el directorio del docker-compose.yml ejecuto el comando kompose convert, pero esto nos creara muchos archivos y lo que queremos hacer es dirigirlo a un solo archivo, para eso usamos la opcion -o de output. El comando quedaria:

``` kompose convert -o ./kompose-output```

Luego utilizamos kubectl para implementar el docker-compose convertido con el kompose.

``` kubectl apply -f ./kompose-output```

Si al momento de correr kubectl aparece el error 
> The connection to the server localhost:8080 was refused - did you specify the right host or port? 

Lo mas probable es que no este abierto el puerto 8080, encontre esta solución. Para usar kubeadm necesitas ciertos requerimientos físicos

```
  kubeadm init
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  export KUBECONFIG=/etc/kubernetes/admin.conf

-------- root cause:
 export KUBECONFIG=/etc/kubernetes/admin.conf
the owner of /etc/kubernetes/admin.conf is root.

------- workaround:
 export KUBECONFIG=$HOME/.kube/config
```
