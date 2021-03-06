# docker
Taller con docker

Vamos a iniciar creando un docker con Centos, creando una imagen con un Dockerfile

1.- Iniciaremos trabajando con el dockerfile = https://github.com/agus100cia/docker/blob/master/inicio/Dockerfile

1.1.- Ubicarse en la carpeta donde se encuentra el dockerfile

1.2.- Construir la imagen

````shell
docker build .

`````

Construir con un nombre (tag)

````shell
docker tag . <usuario>/<repoName>:<tagName> --no-cache

Ej: ocker build . --tag amartinez/centos:centos7inicial --no-cache

``````

NOTA: El parametro --no-cache es opcional si queremos que cree la imagen desde cero

1.3.- Ver las imagenes

`````shell
docker images -a

`````

se ve algo asi

agus@MacBook-Pro-de-Agustin inicio % docker images -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              589dc4d40385        42 hours ago        237MB


1.4.- Borrar la imagen se coloca como parametro el IMAGE ID

````shell
docker rmi $(docker images -a -q)

`````
Borrar una imagen de manera forzada (cuando la imagen esta ejecutandose)

````shell
docker rmi -f $(docker images -a -q)

`````

1.5.- Ejecutar 

```shell
docker run -it 08d05d1d5859   bash

````

Parametros: 

-it = Interactivo, es decir cuando arranque podemos interactuar inmediatamente

bash = Se ejecutara la consola del contenedor

1.6.- Cuando la imagen esta ejecutandose se convierte en un CONTENEDOR y tiene su propio ID

Ver los contenedores:

````shell
docker ps  

EJM: 
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
85fd6c595b37        f5833ea08135        "bash"              About a minute ago   Up About a minute                       qu
`````

1.7.- Copiar un archivo del docker a la maquina real

`````shell
docker cp <containerId>:/file/path/within/container /host/path/target

EJM:
docker cp 85fd6c595b37:/etc/elasticsearch/elasticsearch.yml .

``````

1.8.- Iniciar otra terminal en un contenedor

`````shell
docker exec -it <CONTAINER_ID> bash

EJM:
docker exec -it 85449527a91e bash

`````   

1.9.- Matar a un contenedor

````
docker kill <CONTAINER_ID>

EJM:
docker kill 85449527a91e

````

1.10.- Ver todos los Containers

````shell
docker container ls -a
```` 

1.11.- Stop and remove all containers

````shell
docker stop $(docker ps -a -q)

docker rm $(docker ps -a -q)

````

1.12.- Mapear puertos y Volumenes

````shell
docker run -p localport:containerport   -v localfolder:containerfolder --name containerName 

EJM:
docker run \
-p 9200:9200 \
-p 9300:9300 \
-v /Users/agus/Documentos/docker/elasticsearch/data:/usr/share/elasticsearch/data 	\
-v /Users/agus/Documentos/docker/elasticsearch/logs:/usr/share/elasticsearch/logs \
--name myelastic \
-e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.5.1 
`````  

1.13.- Ejecutar docker en Background

-d=true  o tambien solo -d

````shell

docker run \
-d \
-p 9200:9200 \
-p 9300:9300 \
-v /Users/agus/Documentos/docker/elasticsearch/data:/usr/share/elasticsearch/data 	\
-v /Users/agus/Documentos/docker/elasticsearch/logs:/usr/share/elasticsearch/logs \
--name myelastic \
-e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.5.1 

````



