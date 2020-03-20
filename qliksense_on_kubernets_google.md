# Instalar Qliksense en Kubernets sobre Google Cloud.

Se instala Qlik Sense Enterprise on Kubernetes (QSEoK) April 2019 ++

## 1.- Crear un clúster GKE

````` ssh 
gcloud container clusters create qseok-gke --machine-type "n1-standard-2" --num-nodes 2 --zone us-central1-c

``````  

## 2.- Obtener las credenciales de GKE

`````ssh
gcloud container clusters get-credentials qseok-gke --zone us-central1-c

`````

## 3.- Habilitar la autenticación RBAC (para ambientes productivos)

Archivo: rbac-config.yaml

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

````

````ssh
kubectl apply -f rbac-config.yaml

````` 

## 4.- Iniciarl Helm Tiller 

````ssh

helm init --client-only

`````

en caso de no funcionar usar este script para instalar helm

`````sh
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


## 5.- Desplegar un provisionador NFS

Archivo: gke-nfs-values.yaml

Sin RBAC
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

  accessMode: ReadWriteMany
  size: 100Gi

## For creating the StorageClass automatically:
storageClass:
  create: true

  ## Set a provisioner name. If unset, a name will be generated.
  provisionerName: kubernetes.io/gce-pd

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


`````

Con RBAC

``````yaml 

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

  accessMode: ReadWriteMany
  size: 100Gi

## For creating the StorageClass automatically:
storageClass:
  create: true

  ## Set a provisioner name. If unset, a name will be generated.
  provisionerName: kubernetes.io/gce-pd

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


`````sh
helm install -n nfs stable/nfs-server-provisioner -f gke-nfs-values.yaml

`````

## Crear un PVC (opcional para probar que el Provisionador NFS funciona)

Archivo: nfs-vol-pvc.yaml

``````yaml
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

`````sh
kubectl apply -f nfs-vol-pvc.yaml

`````

## Añadir el repositorio de Qlik en Helm

`````sh
helm repo add qlik https://qlik.bintray.com/stable

`````


## Desplegar Qliksense init Chart

`````sh
helm install -n qseok-init qlik/qliksense-init

``````

## Desplegar Qliksense Chart

archivo: qseok-values.yaml

`````yaml 
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

`````sh
helm install -n qseok qlik/qliksense -f qseok-values.yaml

```````

## Obtener la ip externa

`````sh
kubectl get svc -w

``````
