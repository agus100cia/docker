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
docker tag . <usuario>/<repoName>:<version> --no-cache

Ej: ocker build . --tag amartinez/centos:7 --no-cache

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


