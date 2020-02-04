## Cloudera con dockers

## 1.- Descargamos la última imagen de cloudera

````ssh
docker pull cloudera/quickstart:latest
````

## 2.- Levantamos el docker con algunos parámetros

`````ssh
docker run \
-m 4G \
--memory-reservation 2G \
--memory-swap 8G \
--hostname=quickstart.cloudera \
--privileged=true \
-t \
-i \
-v $(pwd):/zaid \
--publish-all=true \
-p8888 \
-p8088 cloudera/quickstart /usr/bin/docker-quickstart
``````

Parámetros:

-m: El docker corre con un maximo de 4Gb en RAM

--memory-reservation: Límite flexible que se activa cuando el docker detecta poca memoria

--memory-swap: Memoria swap

-i: Interactivo es decir que cuando inicie el docker podremos ejecutar alguna aplicación, en nuestro caso sera ssh

-t: Abrirlo en terminal

-v:Comparte volumenes con el contenedor LOCALFOLDER:DOCKERFOLDER

--publish-all: Abre todos los puertos de las aplicaciones tales como HUE 8888, YARN, etc.


## HUE y YARN

HUE: http://localhost:32768  (usuario: cloudera: clave: cloudera)

YARN: http://localhost:32769

