# K8s 

## Conceptos

### Kubectl-proxy  

Es un proxy que corre en foregrown (en userspaces) que nos permite acceder a la API de K8s de manera autenticada, ya que se requiere unas credenciales para acceder con permisos de autenticación.

kubectl-proxy es un proxy que corre foreground y nos permite acceder a la API de kubernetes de manera autenticada.

## kubectl-portforward  

nos permite hacer lo mismo que el proxy pero, para acceder a cualquier puerto de un servicio que este expuesto de nuestro Cluster.

kubectl port-forward nos permite realizar lo mismo que kubectl-proxy, pero accediendo a cualquier puerto del servicio expuesto en nuestro cluster

### Cluster IP

Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.

### NodePort

Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.

## DaemonSet 
se asegura que todos, o algunos nodos corra una copia de un Pod. No se crean en el CLI (kubectl), sino a través de los manifest files.

## ¿Cómo sabe k8s como enviar trafico a que pods?
Lo hace a tráves de los selectores, este permite el balanceo de carga. Cada service tiene los endpoints de los pods que estan corriendo para que otros
servicios puedan acceder. Al eliminar uno, k8s crea otro para cumplir con el estado deseado y lo integra automaticamente para ser destinatario de trafico.

## maxSurge: 
es un campo opcional que indica el número máximo de Pods que pueden crearse por encima del numero deseado de Pods, si tengo 100 Pods y se empieza a actualizar con un 25%, puedo tener en un momento hasta 125 Pods (25 Pods eliminandose y/o 25 Pods creandose y 75 Pods normales).

## maxUnavailable: 
es un campo opcional que indica el número máximo de Pods que pueden no estar disponibles durante el proceso de actualización (valor = int | %, if n=0 then n=25%) si tengo 100 Pods, y se empieza a actualizar la aplicación, con un 25%, tendré 75% de los Pods siempre funcionando. (25 Pods no disponibles)


## Healthchecks 
es un organismo que tiene kubernetes para saber si nuestra aplicación está funcionando de una manera correcta para saber si debe removerla o reiniciarla al no estar en un estado deseable.


## Tenemos diferentes formas de configurar nuestras aplicaciones:

- Argumentos por línea de comandos
- Variables de entorno (env map en el spec)
- Archivos de configuración (config maps)
>>> Guardan tanto archivo como valores clave/valor

## ContHay cuatro maneras diferentes de usar un ConfigMap para configurar un contenedor dentro de un Pod:

- Argumento en la linea de comandos como entrypoint de un contenedor
- Variable de enorno de un contenedor
- Como fichero en un volumen de solo lectura, para que lo lea la aplicación
- Escribir el código para ejecutar dentro de un Pod que utiliza la API para leer el ConfigMap


## Un configmap es:
un objeto de la API utilizado para almacenar datos no confidenciales en el formato clave-valor.

Un ConfigMap te permite desacoplar la configuración de un entorno específico de una imagen de contenedor, así las aplicaciones son fácilmente portables.


## volumen 
nos va a permitir compartir archivos entre diferentes pods o en nuestro host. Estos se usan para que los archivos vivan a lo largo del tiempo y el pod pueda seguir haciendo uso de estos archivos de logs, archivos de configuración o cualquier otro.

### Docker:

>> Permiten compartir información entre contenedores del mismo host
>> Permiten acceder a mecanismo de storage
>> Docker config y docker secrets

### Kubernetes:

Permiten compartir información entre contenedores del mismo pod
Permite acceder también a mecanismo de storage
Se utilizan para el manejo de secrets y configuraciones
Ciclo de Vida

El volumen se crea cuando el pod se crea.
– Esto aplica principalmente para los volúmenes emptyDir.
– Para otro tipo se conectan en vez de crearse.
Un volumen se mantiene aún cuando se reinicie el contenedor.
Un volumen se destruye cuando el pod se elimina.


## Autenticación
es el método por el cual Kubernetes deja ingresar a un usuario.

```kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')```

### Ver service accounts
```kubectl get serviceaccounts```
```kubectl get sa```
### Ver service account por defecto (ver secreto asociado)
```kubectl get sa default -o yaml```
### Obtener token de secreto asociado
```kubectl get sa default -o json | jq -r '.secrets[0].name'```
### Guardar token de secreto asociado en una variable
```K8S_SECRET_NAME=$(kubectl get sa default -o json | jq -r '.secrets[0].name')```
### Ver secreto
```kubectl get secret $K8S_SECRET_NAME -o yaml```
### Decodificar token
```kubectl get secret $K8S_SECRET_NAME -o json | jq -r .data.token | base64 -d```
### Guardar token
```K8S_TOKEN=$(kubectl get secret $K8S_SECRET_NAME -o json | jq -r .data.token | base64 -d)```
### Ver servicios del namespace "default" y copiar la IP del servicio "kubernetes"
```kubectl get svc -n default```
### Acceder al cluster (SOLO MINIKUBE)
```minikube ssh -p multinode-platzi-demo```
### Acceder al servicio de kubernetes con token de autenticación
#### NOTA: Si se utiliza minikube la variable $K8S_TOKEN está en otro entorno, por lo que será necesario ingresarla manualmente
curl -k https://$IP_SERVICIO_K8S -H "Authorization: Bearer $K8S_TOKEN"


## Autorización

es el mecanismo para que un usuario tenga una serie determinada de permisos para realizar ciertas acciones sobre el cluster.

> - Cuando el API server recibe un request intenta autorizarlo con uno o más de uno de los siguientes métodos: Certificados TLS, Bearer Tokens, Basic Auth o Proxy de autenticación.
> - Si cualquier método rechaza la solicitud, se devuelve un 401.
> - Si el request no es aceptado o rechazado, el usuario es anónimo.
> -Por defecto el usuario anónimo no puede hacer ninguna operación en el cluster.

### Role based access control(RBAC) 
es un mecanismo de kubernetes para gestionar roles y la asociación de estos a los usuarios para delimitar las acciones que pueden realizar dentro de la plataforma.

- Un rol es un objeto que contiene una lista de rules
- Un rolebiding asocia un rol a un usuario
- Pueden existir usuarios, roles y rolebidings con el mismo nombre
- Una buena práctica es tener un 1-1-1 bidings
- Los Cluster-scope permissions permiten definir permisos a nivel de cluster y no solo namespace
- Un pod puede estar asociado a un service-account