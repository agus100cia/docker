# docker
Taller con docker

Vamos a iniciar creando un docker con Centos, creando una imagen con un Dockerfile

1.- Iniciaremos trabajando con el dockerfile = https://github.com/agus100cia/docker/blob/master/inicio/Dockerfile

1.1.- Ubicarse en la carpeta donde se encuentra el dockerfile

1.2.- Construir la imagen

````shell
docker build .

`````

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


