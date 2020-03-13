## Generar un pod
## Define el nombre del pod y la imagen base
kubectl run --generator=run-pod/v1 podtest --image=nginx:alpine

## Listar pods
```ssh

kubectl get pods
```
## Listar pods

## Ver los logs del pod
Crear un pod podtest1 de una imagen que no existe, el pod si se crea
Container creating pero no esta ready
```ssh
kubectl describe pod podtest1
```


## Borrar un pod
````ssh 
kubectl delete pod podtest
kubectl delete pod pod1 pod2
````


## YAML desde un pod
YAML define el minimo de replicas en caso de fallas o replicas
Obtener el YAML de un pod
````ssh
kubectl get pod podtest -o yaml
docker ps -l 
docker ps -f=name=hskdjhfksdhfksdhfkd
````


## Obtener la ip de un pod
## Ver las redes
```ssh
kubectl get service --all-namespaces
````


## Entrar al pod
````ssh
kubectl exec -it podtest sh
````
## Entrar al pod

## Ir a la carpeta 

/usr/share/nginx/html

##Logs

````ssh 
kubect logs podtest -f
````

##Implementar un pod desde un yaml

````ssh
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml

kubectl get pod shell-demo

kubectl exec -it shell-demo -- /bin/bash



root@shell-demo:/# ls /
root@shell-demo:/# ls /
root@shell-demo:/# cat /proc/mounts
root@shell-demo:/# cat /proc/1/maps
root@shell-demo:/# apt-get update
root@shell-demo:/# apt-get install -y tcpdump
root@shell-demo:/# tcpdump
root@shell-demo:/# apt-get install -y lsof
root@shell-demo:/# lsof
root@shell-demo:/# apt-get install -y procps
root@shell-demo:/# ps aux
root@shell-demo:/# ps aux | grep nginx

root@shell-demo:/# echo Hello shell demo > /usr/share/nginx/html/index.html

oot@shell-demo:/# apt-get update
root@shell-demo:/# apt-get install curl
root@shell-demo:/# curl localhost

`````



