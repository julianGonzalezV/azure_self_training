Chapter 1
# Development Process--->

# Microservice 
- retry logic or circuit breakers is critical to avoid a single fault causing application
downtime.

# Devops 
- CI: Tool used to automated manual tasks -- testing---build code
- CD: tools used to automate the delivery/deployment of the applications 

# Operation phase
When a piece of software is running 


# Kubernetes 
Container orchestration platform (what happen when millions of cotainers exist? how to manage them? R Kubernetes!)
it takes care of:
	scheduling containers to be run on servers
	restarting containers when they fail
	containers to a new host when that host becomes unhealthy
	much more ..
	
Note: Although Kubernetes supports multiple container runtimes, Docker is the most
popular runtime.

## PODS: 
is the essential scheduling block.contains either a single container or multiple containers.(you can use the terms container and Pod
interchangeably. However, the term Pod is still preferred.)
-Un Pod es efimero (se puede terminar stop y al reinicarse en otro nodo)..se pierde el estado!(debe almacenarlo en Bds externas o en un statefulSet)

Pod Design:
    XXX: Tener un contenedor por cada Pod (común en xxx)
	Sidecar : describe que se puede tener multiples container en un POD peero que sean altamente acoplado(casi que el mismo contexto de negocio más algo adicional referente a la mismo!)
		Ejemplo -> 2 container en un Pod 1-para el app web que no soporta https y el 2do para colocarle seguridad(SSL offloading)
		entre 2 y 1 se comunican vía http peero 2 expone la seguridad y es el punto de entrada del POD
		> La comunicación entre containers en un Pod se vía localhost


# Deployments in Kubernetes
- A deployment is a Layer around Pods(to create, update, scaling and autoSacaling the applications), difeines the desire state of the application
## Replica(copy) Set: 
- Is an object in Kubernetes that guarantees that certain number  of PODS will be ALWAYS available 
- A deployment ceates a REPLICASET, this replica creates the Pods

Deployment --> ReplicaSet--->Pods 
(Relationship)


# Service
-Is an object that provides a static IP and DNS name to an application
-Network level abstraction to expose the Pods under a single IP and DNS : Cada pod tiene su propia ip cuando se crean ..usted puede conectarlos haciendo llamadoss entre ellos(por sus Ips)
Peeero recuerde que los Pods son efimeros y al volver a crearsen sus IPs cambian ...por eso necesita el Service , este se encarga de mantener la conexión enre Pods, asi estos  mueran y nazcan 

# ConfigMap :
Objeto to provide configuration details to PODs: dado que es posible tener estas conf fuera del contenedor, es viable proveer dichas configuraciones directamente 
conectandolo al objeto DEPLOYMENT

# Helm:
Package manager for Kubernetes: Simplifica el proceso de despliegue 

# AKS - Azure Kubernetes Service (PaaS)
-To create and manage kubernetes cluster easier
-Contiene la logica para acualizar la versoin de kubernetes a las más nuevas
-Permite scalar facilmente los clusters para hacerlos más grandes o pequeños
-Viene con una pre-configuracion de azure AD/active directory en RBAC/role bases access control , para crear reglas de acceos a ls recursos 
-Al levantar un cluster con AKS, se crean nodos(virtual machines) master(bd y ap para almacenar el estado del cluster ) 
y workers(nuestras aplicaciones), solo se paga por los workers y no los primeros 
-Services in AKS están integrados con ALB/Azure load Balancer --> layer4 into Open Systems Interconnection (OSI) model
Kubernetes Ingress en AKS están integrados con ApiGateway ---> layer7 into Open Systems Interconnection (OSI) model
Lo anterior quiere decir que cuando se cree un Service o Ingress en Kubernetes entonces se creará una regla en ALB o en el application Gateway, y algunos de estos 2
enrrutarán el trafico al nodo(s) correcto  del cluster que contiene los Pods
-Aks 
az aks show --name 

az aks show --name aks-hansonaks --resource-group rg-handsonaks

Stop aks y evitar costos
az aks stop --name aks-hansonaks  --resource-group rg-handsonaks

Start Aks:
az aks start --name aks-hansonaks  --resource-group rg-handsonaks


Chapter 2:
# Kubernetes on azure ---> AKS 

## ways to deploy an Aks
1 Portal
2 Azure Cli
3 azure PowerShell
5 ARM Azure resource manager (native azure IaC)---> validar con david si es mejor
6 Terraform for azure --https://docs.microsoft.com/azure/aks 


# Creating aks via Portal
See books\process-images folder images named from aks-01 --> aks-09

# Accessing your cluster using Azure Cloud Shell
See books\process-images folder images named from aks-10 --> aks-11
starting commands

Para obtener el acceso al cluster (note que rg- es el nombre del resource group y aks- el del kubernetes service)
$ az aks get-credentials --resource-group rg-handsonaks --name aks-hansonaks

Verificar que nos podemos conectar al cluster AKS
$ kubectl get nodes

# Deploying your first demo application
descargar el código en la carpeta deseada (en este ejemplo para llame jag-handsonaks)
git clone https://github.com/PacktPublishing/Hands-On-Kubernetes-on-Azure---
Second-Edition.git jag-handsonaks 

ir al folder recien creado 
cd jag-handsonaks
$ cd Chapter02
$ ls
(you should see the next files: azure-vote-redis-all-in-one.yaml  azure-vote.yaml  docker-compose.yaml)

Para abrir  el archivo .yaml en un editor (para no pelear con vi por ejemplo :) ) el shell digite
$ code azure-vote.yaml
Se le abrirá el editor de texto y podrá hacer los cambios deseado (en la parte superior derecha encontrará tres puntos ... para que guarde, cierre el archivo)
Para verificar que su edición se halla ralizado puede verificar imprimiendo las lienas del archivo 
$ cat azure-vote.yaml

Y AHORA PARA LANZAR LA APLICACION...
kubectl create -f azure-vote.yaml
Salida del comando:
deployment.apps/azure-vote-back created
service/azure-vote-back created
deployment.apps/azure-vote-front created
service/azure-vote-front created

Ver el progreso del lanzamiento 
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
azure-vote-back-d6ddb4dc5-sqvqx     1/1     Running   0          51s
azure-vote-front-7d599865c9-9jwpf   1/1     Running   0          51s

note como el status ya es runing!

para evitar estar ejecutando el comando anterior para ver si sus pods están arriva entonces deje el shell escuchando con --watch :
$ kubectl get pods --watch 
NAME                                READY   STATUS    RESTARTS   AGE
azure-vote-back-d6ddb4dc5-sqvqx     1/1     Running   0          51s
azure-vote-front-7d599865c9-9jwpf   1/1     Running   0          51s

(Note: con ctrl+C lo para )


Para obtener la ip publica del Load balancer (de nuevo watch para cuando se demora en subir)
$ kubectl get service azure-vote-front --watch
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.189.243   20.62.154.9   80:32114/TCP   6m8s


si copia el external Ip en su navegador debería ver la aplicacion desplegada (back y front!!!) (http://20.62.154.9/)


clean up our deployment
 $ kubectl delete -f azure-vote.yaml
 Salida:
deployment.apps "azure-vote-back" deleted
service "azure-vote-back" deleted
deployment.apps "azure-vote-front" deleted
service "azure-vote-front" deleted






