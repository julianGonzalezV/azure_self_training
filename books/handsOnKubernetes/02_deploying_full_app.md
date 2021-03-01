hasta pg92
# Deploying second  application :) guestbook application
julian@Azure:~/jag-handsonaks$ cd Chapter03

julian@Azure:~/jag-handsonaks/Chapter03$ ls

example-redis-config.yaml  mariadb_statefulset.yaml  redis-config.yaml                     redis-master-deployment.yml  redis-slave-service.yaml
frontend-deployment.yaml   nillsf.yaml               redis-master-deployment_Modified.yml  redis-master-service.yaml
frontend-service.yaml      redis-config              redis-master-deployment.yaml          redis-slave-deployment.yaml
julian@Azure:~/jag-handsonaks/Chapter03$

deploy redis master --(more about redis master-slave see https://redis.io/topics/replication)
Understandign YAM File-See page 69 on Hands-On-Kubernetes-on-Azure book 
$ kubectl apply -f redis-master-deployment.yaml

Verificar componentes creados de ejecutar el comando 
$ kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/redis-master-7db7f6579f-42248   1/1     Running   0          9h

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   17h

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-master   1/1     1            1           9h

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-master-7db7f6579f   1         1         1       9h

Interesante! , note que se creó un deployment llamado deployment.apps/redis-master, este a su vez crea un replica set replicaset.apps/redis-master-7db7f6579f
, el replica a su vez crea el pod named pod/redis-master-7db7f6579f-42248  (note como el nombre de cada objeto va agregando el name id de su objeto padre)


el NAME service/kubernetes es el servicio de kubernetes levantado al crear el aks!

Después de la práctica entonces clean up :) 
$ kubectl delete deployment/redis-master

# Deployment using ConfigMap 
-La caracteristica adicional que nos brinda ConfigMap es que no se requiere Crear una imagen docker por que nocesitemos 
-En la practica por lo regular se hacen despliegues que contienen configuraciones
-El ConfigMap contiene key-value pair para datos que se requieren agregar a un contenedor---OJO es SOLO para non-secret
 configuration para ello hay otro objeto en kubernetes llamado SECRET 
# Two ways to create ConfigMap file

## 1 Creating a ConfigMap from a file
crear un archivo sin exptensión y colocar las key-value pairs ejemplo 

maxmemory 2mb
maxmemory-policy allkeys-lru

crear el ConfigMap tomando como archivo origen el creado anteriormente (para este caso se creó el archivo con nombre jag-config)
$ kubectl create configmap jag-config --from-file=jag-config 

Para verificar la creación del ConfigMap entonces ejecute 
$ kubectl describe configmap/jag-config
Salida en consola
***************************************
Name:         jag-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
jag-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
Events:  <none>
***************************************

NOTA: Para terminar el ejemplo :) 
$ kubectl delete configmap/jag-config

## 2 Creating a YAML file
Cree el archivo .yaml con las configuraciones que requiera: (por comando desde la consola azure eg code jagyaml-config.yaml )
Note que los key-val de configuracion van el en redis-config
jagyaml-config.yaml
---------------------------
apiVersion: v1
data:
    redis-config: |-  (nombre de la configuracion, cambielo para que aplique a su proyecto)
        maxmemory 2mb
        maxmemory-policy allkeys-lru
kind: ConfigMap
metadata:
    name: jag-yml-config  ---OJO este es el nombre con que quedará el ConfigMap
    namespace: default
---------------------------
cree el ConfigMap
$ kubectl create -f jag-config.yaml
Verifique su creacion 
$  kubectl describe configmap/jag-yml-config (nombre en el metadata del archivo anterior!)
Name:         jag-yml-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
Events:  <none>

NOTA:
use el sgte comando para ver el yaml(-o yaml) o json (-o json) del ConfigMap existente, solo lo imprime en pantalla ..sea curioso y vea como lo escribe en otro archivo :) 
$ kubectl get -o yaml configmap/jag-yml-config


#Using a ConfigMap to read in configuration data
See page 76 para explicacino de archivo redis-master-deployment.yaml y vea las diferencias con redis-master-deployment-mod.yaml

## Create a deployment base on redis-master-deployment-mod.yaml ---este es el BACKEND
$ kubectl create -f redis-master-deployment_Modified.yml

Verify changes 
$kubectl get pods

SalidaNAME                            READY   STATUS    RESTARTS   AGE
redis-master-79695874f8   1/1     Running   0          13s

Un solo Pod master!

Exec into the pod and verify settings were applied
$kubectl exec -it redis-master-<pod-id>  --redis-cli
en este ejemplo sería 
 kubectl exec -it redis-master-79695874f8-qj8l6 -- redis-cli
127.0.0.1:6379> CONFIG GET
(error) ERR Wrong number of arguments for CONFIG GET
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"

#delete a deployment
$kubectl delete deployment/redis-master

## Create The service:
Recuerde que el service es para exponer los pods vía label match, ya que los pods por so solos ya tienen una IP pero es dinamica 
cada vez que se reinician o crean 
Pg80 --> explicacion del archivo redis-master-service.yaml -->(recuerde que redis puede cambiar por el nombre de su proyecto --obio cierot :) )

Create:
$ kubectl apply -f redis-master-service.yaml
Verify:
$ kubectl get service
salida 
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.0.0.1     <none>        443/TCP    28h
redis-master   ClusterIP   10.0.29.33   <none>        6379/TCP   46s

la ips solo es visible dentro del cluster por eso se llama CLUSTER-IP 

El Service también registra un registro DNS, lo puede verificar Probando ping al service 
$ kubectl exec -it redis-master-79695874f8-qj8l6 -- bash
[ root@redis-master-79695874f8-qj8l6:/data ]$ ping redis-master
PING redis-master.default.svc.cluster.local (10.0.29.33) 56(84) bytes of data.

DNS = redis-master.default.svc.cluster.local y resuelve a --> 10.0.29.33

#Deploying the Redis slaves
Desplegar un solo nodo master no es recomendado por lo cual en ambientes funcionales por lo regular se tienen multiples nodos esclavos PARA LECTURA y 1 master
PARA ESCRITURA 
Create deployment
$ kubectl apply -f redis-slave-deployment.yaml

Check all resources are created
$ kubectl get all
Salida:
NAME                                READY   STATUS    RESTARTS   AGE
pod/redis-master-79695874f8-qj8l6   1/1     Running   0          20h
pod/redis-slave-5bdcfd74c7-7mmss    1/1     Running   0          84m
pod/redis-slave-5bdcfd74c7-lchx2    1/1     Running   0          84m

NAME                   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/kubernetes     ClusterIP   10.0.0.1     <none>        443/TCP    47h
service/redis-master   ClusterIP   10.0.29.33   <none>        6379/TCP   19h

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis-master   1/1     1            1           20h
deployment.apps/redis-slave    2/2     2            2           84m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-master-79695874f8   1         1         1       20h
replicaset.apps/redis-slave-5bdcfd74c7    2         2         2       84m

Note el 2/2 redis-slave
pod/redis-slave-5bdcfd74c7-7mmss   
pod/redis-slave-5bdcfd74c7-lchx2    
Lo anterior confirma las 2 replicas(2 pods) del redis-slave 

#Expose the Redis slaves
$ kubectl apply -f redis-slave-service.yaml
To verify: 
$ kubectl get service
> Hasta acá se tienen un redis cluster con un solo pod master y 2 pods replicas


#Deploying a Front end application

Salida:
NAME                            READY   STATUS    RESTARTS   AGE
frontend-6cb7f8bd65-c5xh2       1/1     Running   0          42s
frontend-6cb7f8bd65-lv9bv       1/1     Running   0          42s
frontend-6cb7f8bd65-m4g6v       1/1     Running   0          42s
redis-master-79695874f8-qj8l6   1/1     Running   0          21h
redis-slave-5bdcfd74c7-7mmss    1/1     Running   0          119m
redis-slave-5bdcfd74c7-lchx2    1/1     Running   0          119m

Note las 3 replicas del front :)
#Expose the Front end

Lo importante es que sepa que hay diferentes TIPOS DE SERVICE, acuerdo al tipo de exposición requerido:
- ClusterIP: Al parecer es el tipo por defecto(verficar :) ), acá los pods son alcanzables por una unica IP que expone el service  
y que solo se puede consumir desde el CLUSTER, no es publico
-NodePort: El service se expone mediante un Puerto estatico que le asigna a cada Pod y es público, pero cada pod tienen un puerto diferente
-LoadBalancer: Acá al crear el service se crea un Azure Load Balancer que expone publicamente una IP, que es la misma para todos los PODS, 
realmente el load balancer sabe como distribuir los llamados que se le hagan 

Mas info en el archivo frontend-service.yaml

Ahora si deploy :)
$ kubectl create -f frontend-service.yaml
To verify 
$ kubectl get service
salida:
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
frontend       LoadBalancer   10.0.110.81    52.151.214.33   80:30118/TCP   2m6s
kubernetes     ClusterIP      10.0.0.1       <none>          443/TCP        2d1h
redis-master   ClusterIP      10.0.29.33     <none>          6379/TCP       20h
redis-slave    ClusterIP      10.0.105.227   <none>          6379/TCP       81m

Note el tipo del service frontends --> LoadBalancer!

ad0bbe10239ec4e98924b35ad8b55947-TCP-80 (Tcp/80)

**********************************
Por último BORRAR todo lo creado

$ kubectl delete deployment frontend redis-master redis-slave
$ kubectl delete service frontend redis-master redis-slave
**********************************