# Qliksense en Kubernetes 


## Requerimientos:

* Google Cloud utilizando Motor de Google Kubernetes (GKE)
* Kubernetes clúster mayor que v1.10.xy menor que v1.16.x
* Processors (CPUs)	Minimum 4 cores
* Memoria minima: 16Gb
* Espacio en disco 100Gb

![img](https://github.com/agus100cia/docker/blob/master/qliksense-kubernets-gcloud.png)

## Crear un cluster:

````ssh
gcloud container clusters create qseok-gke --machine-type "n1-standard-2" --num-nodes 2 --zone us-central1-c

````` 

Crear un archivo rbac-config.yaml

`````yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system

``````   

````ssh
kubectl apply -f rbac-config.yaml
helm init --service-account tiller

````` 

Archivo gke-nfs-values.yaml

`````yaml 
# Default values for nfs-provisioner.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

# imagePullSecrets:

image:
  repository: quay.io/kubernetes_incubator/nfs-provisioner
  tag: v2.2.1-k8s1.12
  pullPolicy: IfNotPresent

service:
  type: ClusterIP

  nfsPort: 2049
  mountdPort: 20048
  rpcbindPort: 51413
  # nfsNodePort:
  # mountdNodePort:
  # rpcbindNodePort:

  externalIPs: []

persistence:
  enabled: true

  ## Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  storageClass: "standard"

  accessMode: ReadWriteOnce
  size: 100Gi

## For creating the StorageClass automatically:
storageClass:
  create: true

  ## Set a provisioner name. If unset, a name will be generated.
  # provisionerName:

  ## Set StorageClass as the default StorageClass
  ## Ignored if storageClass.create is false
  defaultClass: false

  ## Set a StorageClass name
  ## Ignored if storageClass.create is false
  name: nfs-dynamic

  # set to null to prevent expansion
  allowVolumeExpansion: true
  ## StorageClass parameters
  parameters: {}

  mountOptions:
    - vers=4.1
    - noatime

  ## ReclaimPolicy field of the class, which can be either Delete or Retain
  reclaimPolicy: Delete

## For RBAC support:
rbac:
  create: true

  ## Ignored if rbac.create is true
  ##
  serviceAccountName: default

resources: {}
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  # requests:
  #  cpu: 100m
  #  memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}

`````` 

````ssh
helm install -n nfs stable/nfs-server-provisioner -f gke-nfs-values.yaml

`````  
Archivo nfs-vol-pvc.yaml

`````yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: qseok-vol
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: "nfs-dynamic"
  resources:
    requests:
      storage: 5Gi
      
``````  

`````ssh
kubectl apply -f nfs-vol-pvc.yaml
helm repo add qlik https://qlik.bintray.com/stable
helm install -n qseok-init qlik/qliksense-init

````` 


Archivo qseok-values.yaml

``````yaml 
devMode:
  enabled: true
engine:
  acceptEULA: "yes"
global:
  persistence:
    storageClass: "nfs-dynamic"
    
identity-providers:
  secrets:
    idpConfigs:
      - discoveryUrl: "https://dev-test.eu.auth0.com/.well-known/openid-configuration"
        clientId: "masked"
        clientSecret : "masked"
        realm: "Auth0"
        hostname: "elastic.example"
        claimsMapping:
          client_id: [ "client_id", "azp" ]
          groups: "/https://~1~1qlik.com~1groups"
          sub: ["/https:~1~1qlik.com~1sub", "sub"]
          
          
``````   

`````ssh
helm install -n qseok qlik/qliksense -f qseok-values.yaml
kubectl get svc -w

``````  

https://support.qlik.com/articles/000082465



## Instalación:

Qliksense enterprise sobre Kubernets es un conjunto de imágenes concentradas en un paquete de Helm Chart (Herramienta para la administración de Kubernets).
El clúster de Kubernets debe tener acceso al FileStorage para persistir los datos.

Se requiere tener instalado:
* Kubectl
* Helm: Es un administrador de paquetes creado para Kubernets. Es como un MAVEN en Java.

Fuente: https://medium.com/google-cloud/installing-helm-in-google-kubernetes-engine-7f07f43c536e

## Instalar Helm sobre un Clúster de Kubernets en Google Cloud

`````ssh
#!/usr/bin/env bash

echo "install helm"
# installs helm with bash commands for easier command line integration
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
# add a service account within a namespace to segregate tiller
kubectl --namespace kube-system create sa tiller
# create a cluster role binding for tiller
kubectl create clusterrolebinding tiller \
    --clusterrole cluster-admin \
    --serviceaccount=kube-system:tiller

echo "initialize helm"
# initialized helm within the tiller service account
helm init --service-account tiller
# updates the repos for Helm repo integration
helm repo update

echo "verify helm"
# verify that helm is installed in the cluster
kubectl get deploy,svc tiller-deploy -n kube-system

``````

Para comprobar que helm se ha instalado correctamente se debe ejecutar

`````ssh
helm ls
`````

Y no debe dar ningun resultado.

## Prepara las herramientas.

Una vez que hemos creado nuestro cluster de Kubernets y además hemos instalado Helm. Debemos:

* Vincular kubectl con Kubernets
* Añadir el repositorio de Qlik helm chart
* Iniciar helm para que trabaje sobre Kubernets

### Vincular kubectl con Kubernets

1.- Verifica que kubectl esta apuntando a kubernets usando el comando

````ssh
kubectl config current-context

`````

2.- Si kubectl no esta apuntando al clúster, obten el listado de todos los clúster disponibles

`````ssh
kubectl config get-clusters

`````` 

3.- Selecciona el clúser con el que vas a trabajar

`````ssh
kubectl config set-cluster <cluster-name>
`````   
### Agregar el repositorio Qlik de helm chart

1.- Agregar el repositorio

`````ssh
helm repo add qlik https://qlik.bintray.com/stable

`````  

2.- Para verificar obten en listado de todos los repositorios configurados

``````ssh
helm repo list

``````   

Para poder usar Helm con tu clúster de Kubernets es necesario:

Añadir Helm Tiller pod al clúster:

````ssh
helm init --wait

````` 

Si tu clúster tiene una seguridad del tipo RBAC (no es necesario en Google Cloud), es necesario añadir estos comandos

`````ssh
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm init --upgrade –-wait
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

````` 


## Instalar Qliksense en Kubernets

Una vez que hemos instalado y configurado kubectl, helm y sus repositorios, estamos listos para instalar Qliksense.

Para instalar un ambiente productivo es importante instalar un IDP (autenticación de usuarios) y una base de datos MongoDB.

Es conveniente disponer de un archivo value.yaml, para poder instalar y configurar los parámetros necesarios.


### 1.-Crea un archivo llamado values.yaml

`````yaml
#Estos parametros de configuracion habilita el modo developer el cual incluye una base local de MongoDB
devMode:
  enabled: true

#Este parámetro acepta el EULA (End-User License Agreement) para el producto
engine:
  acceptEULA: "yes"
  
#Este parámetro especifica los servicios de almacenamiento
#El parametros StorageClass debe ser tomado del cluster en el caso de google es standard
global:
  persistence:
    storageClass: standard  

``````

Qliksense creará motores para las recargas programadas. Para poder hacer esto el Engine debe configurarse en Kubernets como un recurso personalizado.
Este paso se debe realizar una vez por clúster.

`````ssh
helm install --name qliksense-init qlik/qliksense-init

`````  

Ahora vamos a instalar Qliksense, esto incluye descargase las imágenes.

````ssh
helm install -n qliksense qlik/qliksense -f values.yaml

````` 

Para verificar el progreso del comando usamos:

``````ssh
kubectl get pods

``````  
Si la implementación fue exitosa, se verán a todos los pods en modo Running.

## Accediendo a Qliksense

Debemos obtener la IP donde esta ejecutándose el servicio. Para ello ejecutamos

`````ssh
kubectl describe service qliksense-nginx-ingress-controller

`````

## Borrar todos los pods

````ssh
kubectl delete --all pods
helm del --purge qliksense;
kubectl delete all --all

```` 





