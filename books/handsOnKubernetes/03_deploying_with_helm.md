desde pg93-104
Helm: Package manager for kubernetes to deploy,update and manage kubernetes applications at scale via writting Helm Charts 
Helm Charts son archivos Yaml(como los usados en 02_deploying_full_app ) pero esta vez no son estáticos (si deseamos cambiar algo debemos entrar al archivo y hacerlo)
sino que permite parámetros que se pueden enviar dinámicamente(permiten mayor escalabilidad), util por ejemplo en despliegues más complejos que requieren que 
le especifiquemos el ambiente(dllo, test, prod)

Para más info acerca de como escribir helm charts :
https://blog.nillsf.com/index.php/2019/11/23/writing-a-helm-v3-chart/

# Use case: Installing WordPress using Helm
- Muchas veces no es necesario escribir los propios helm charts, ya que Helm ofrece una variedad de librerías con Helm charts pre-escritos
para este ejemplo se usara el helm chart pre-escrito para wordpress
para ello:
1-adicionar el repo de helm charts 

$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/

2 Instalar el chart que se requiera

$ helm install handsonakswp stable/wordpress

To verify;
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
handsonakswp-credentials-test            0/1     Error     0          4m8s
handsonakswp-mariadb-0                   1/1     Running   0          4m8s
handsonakswp-mariadb-test-co8yk          0/1     Error     0          4m8s
handsonakswp-wordpress-8f966546d-62d7b   1/1     Running   0          4m8s


Note como el Pod handsonakswp-mariadb tienen un númerso asociado que no es randomico como wordpress y es porque se introduce un oljeto nuevo de kubernetes llamado 
statefulsets
#statefulsets
Son muy similares a deployments(que contienen staless applications ) pero que maneja la singularidad de cada Pod y asegura que un pod esté junto a 
su storage(entiendo que es donde se alamacena el estado) en un mismo lado
$ kubectl get statefulsets
Para ver que contiene el yaml del stateful
$ kubectl get statefulset -o yaml > mariadbss.yaml
$ code mariadb.yaml pg 96

$kubectl get storageclass
$ helm ls
$ helm status handsonakswp

Clean up del ejercio
helm delete handsonakswp

Borrar el PVC_Persistent volumen claim 

kubectl delete pvc data-handsonakswp-mariadb-0
kubectl delete pvc  handsonakswp-wordpress--no fué necesario correrlo , al parecer con ejecutar el helm delete handsonakswp lo tomó
