# Qliksense en Kubernetes 


## Requerimientos:

* Google Cloud utilizando Motor de Google Kubernetes (GKE)
* Kubernetes clúster mayor que v1.10.xy menor que v1.16.x
* Processors (CPUs)	Minimum 4 cores
* Memory	Minimum 8 GB
* Disk space	5 GB total required to install

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
````


