## Instalar helm
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

## Verificar si tenemos helm instalado
helm

## Configurar helm
helm init

## Verificar si Tiller está instalado
kubectl get pods -n kube-system

## Dar permisos a helm
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default

## Buscar paquetes
helm search

## Ejemplo de cómo instalar un paquete
helm search prometheus
helm inspect stable/prometheus | less
helm install stable/prometheus --set server.service.type=NodePort --set server.persistentVolume.enabled=false

## Obtener servicios

kubectl get svc

## Crear helm chart
helm create dockercoins
cd dockercoins
mv templates/ templates-old
mkdir templates
cd ..
kubectl get -o yaml --export deployment worker