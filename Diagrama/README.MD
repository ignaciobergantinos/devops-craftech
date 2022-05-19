#### Prueba 1 - Diagrama de Red

Produzca un diagrama de red (puede utilizar lucidchart) de una aplicación web en GCP o AWS y escriba una descripción de texto de 1/2 a 1 página de sus elecciones y arquitectura.

El diseño debe soportar:

- Cargas variables
- Contar con HA (alta disponibilidad)
- Frontend en Js
- Backend con una base de datos relacional y una no relacional
- La aplicación backend consume 2 microservicios externos

 
El diagrama debe hacer un mejor uso de las soluciones distribuidas.


## Solución:

Para esta solución vamos a utilizar la nube de AWS.

![Diagrama AWS](devops-craftech.drawio.png)

Como se trata de un ejercicio de contenedores, primero hay que decidir que servicio usar para este requerimiento. Podemos usar: AWS EC2, AWS ECS (Nodos EC2 o Fargate), AWS Elastic Beanstalk, o EKS. Nuestra decisión va a variar según el contexto en el que nos encontremos:

- AWS EC2: Bajo Precio, poca eficiencia
- Elastic Beanstalk: Poca complejidad (Alto nivel de abstracción)
- EKS: Requerimientos multi-nube 
- ECS: Modo EC2 (Eficiente) / Fargate (Serverless)

Se utilizará **AWS Elastic Container Service (ECS): Modo EC2** para este ejercicio, y la razón es simplemente utilizar instancias ec2 en un cluster, olvidandonos de precio-complejidad.

Nuestro diseño necesita soportar **cargas variables**, para esto se utilizarán Auto Scaling Groups, que permitirán que nuestras instancias EC2 dentro del cluster ECS usen el concepto de elasticidad, cuando la demanda aumente, se lanzarán nuevas instancias, y en caso de que la demanda decrezca, se terminarán instancias. En el caso de las instancias de Aurora, se escalan automaticamente. Y en el caso de DynamoDB hay que setear el Auto Scaling "ON" 

Tambien nos pide **HA (High Availability)**, con lo cual, replicaremos nuestro diseño a otra zona de disponibilidad, y utilizaremos un Aplication Load Balancer para distribuir la carga entre Clusters ECS de diferentes AZs, usamos un ALB ya que son http/s requests, y eso pertenece a la capa 7 del modelo OSI. 
Si aun queremos más HA, podriamos implementar este diseño a otra región, y utilizar Route 53 para hacer un Active-Pasive Failover. Lo que hace Route 53 es utilizar resource health checks, para determinar si un servicio esta funcionando o no, en el caso que todos los recursos primarios fallen, se redirigirá el trafico a una a un recurso secundario, es decir esto ocurrirá si la region A falla.
En el caso de DynamoDB, hay que utilizar tablas globales, de esta manera tendremos toda la data duplicada en otra region. Para la base de datos relacional, tenemos dos opciones, RDS o Auora. Se eligió Aurora sobre RDS, porque RDS no permite hacer un failover automatico entre regiones, más bien es manual. En el caso de Aurora, promueve una Read Replica automaticamente a una Master replica en el caso de un fallo de region. Si fuese solo una región se podria usar RDS con una implementacion Multi-AZ

Los contenedores soportan **javascript** y permiten interactuar con **Aurora y DynamoDB, bases de datos relacional y no relacional respectivamente**. Por ultimo tambien desde un contentedor se pueden hacer **requests a APIs externas, para el uso de microservicios externos**

Aclaraciones:
- La Aurora Master replica sincronicamente a la Aurora Multi Master, y ambas tienen la capacidad de lectura/escritura
- DynamoDB esta fuera de la VPC, dentro de la región, no es un servicio global
- Aurora acepta PostreSQL