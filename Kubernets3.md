##Generar un pod
##Define el nombre del pod y la imagen base
kubectl run --generator=run-pod/v1 podtest --image=nginx:alpine

## Listar pods
kubectl get pods

## Ver los logs del pod
##Crear un pod podtest1 de una imagen que no existe, el pod si se crea
##Container creating pero no esta ready
kubectl describe pod podtest1


##Borrar un pod
kubectl delete pod podtest
kubectl delete pod pod1 pod2


##YAML desde un pod
##YAML define el minimo de replicas en caso de fallas o replicas
##Obtener el YAML de un pod
kubectl get pod podtest -o yaml
docker ps -l 
docker ps -f=name=hskdjhfksdhfksdhfkd


##Obtener la ip de un pod
## Ver las redes
kubectl get service --all-namespaces


##Entrar al pod
kubectl exec -it podtest sh

## Ir a la carpeta 
/usr/share/nginx/html

##Logs
kubect logs podtest -f


