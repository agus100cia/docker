# Prometeus

Prometheus es parte de un ecosistema que proporciona alertas , paneles de control y un conjunto completo de herramientas para vincularse a sus fuentes de datos existentes

Para instalar Prometheus en Docker , necesitará tener derechos de sudo en su host.

Si no está seguro, ejecute el siguiente comando

```sh
sudo -v
```

Antes de pasar a la siguiente sección, asegúrese de tener la última versión de Docker en ejecución.

```sh
docker --version
Docker version 19.03.1, build 74b1e89
```

## Instalación de Prometeus con Docker

La imagen oficial de prometeus en docker es:

https://hub.docker.com/r/prom/prometheus

La imagen de Prometheus Docker instalará el servidor Prometheus. Es el responsable de agregar y almacenar métricas de series de tiempo.

Los archivos de configuración y los directorios de datos se almacenarán en el sistema de archivos del host local.

Como consecuencia, esas carpetas se asignarán al contenedor de Docker para garantizar la persistencia de los datos. También es más fácil modificar archivos en el sistema de archivos local que en un contenedor.

Para ver las imagenes que tenemos:

```sh
docker image ls

[naadbd01@quisrvbigdata9 conf]$ docker image ls
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
prom/prometheus             latest              6d6859d1a42a        13 days ago         169MB

```

Importar la imagen

```sh
docker pull prom/prometheus
```


### Preparar el ambiente

Cree un usuario para Prometheus en su sistema si aún no existe.

```sh
sudo useradd -rs /bin/false prometheus
mkdir -p /home/naadbd01/docker/prometheus/conf
touch /home/naadbd01/docker/prometheus/conf/prometheus.yml
sudo mkdir -p /home/naadbd01/docker/prometheus/data
sudo chown -R prometheus:prometheus /home/naadbd01/docker/prometheus
```

De esta manera, solo Prometheus y root pueden modificar este archivo. Esto proporciona una mayor seguridad a nuestra configuración de Prometheus.

### Configurar Prometeus

```sh
sudo nano /home/naadbd01/docker/prometheus/conf/prometheus.yml

```

Pegue esta configuracion

```sh
# my global config
global:
  scrape_interval:     15s # Escanea cada 15 segudos. Default is every 1 minute.
  evaluation_interval: 15s # Evalua las reglas cada 15 segundos. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9005']

```

Por ahora, Prometheus solo se va a monitorear a sí mismo.

Más adelante, agregaremos el Node Exporter que será responsable de recopilar métricas de nuestro sistema Linux local.

Ahora que Prometheus está configurado para Docker, vamos a ejecutar el contenedor Prometheus.

### Ejecución de Prometeus

Vamos a utilizar la imagen de la imagen de Prometheus del repositorio de prom.

Asegúrese de que no se esté ejecutando nada en el puerto 9090.

```sh
sudo netstat -tulpn | grep 9005
```
 Vamos a necesitar conocer el ID del usuario prometeus, Esto es lo que usaremos para crear nuestro contenedor Prometheus.
 
 ```sh
id prometheus
```

o tambien

 ```sh
cat /etc/passwd | grep prometheus
prometheus:x:991:977:/home/prometheus:/bin/false

  ```
  
  Para iniciar Prometheus en Docker, ejecute el siguiente comando.
  
 ```sh
 
docker run -d -p 9005:9005 --user 991:977 \
--net=host \
-v /home/naadbd01/docker/prometheus/conf/prometheus.yml:/etc/prometheus/prometheus.yml \
-v /home/naadbd01/docker/prometheus/data:/data/prometheus \
prom/prometheus \
--config.file="/etc/prometheus/prometheus.yml" \
--storage.tsdb.path="/data/prometheus"

```
  
 Aquí hay una explicación de todos los parámetros proporcionados en este comando.
  
- -d : significa modo independiente . El contenedor se ejecutará en segundo plano. Puede tener un shell para ejecutar comandos en el contenedor, pero salir de él no detendrá el contenedor.
- -p : significa puerto . Dado que los contenedores se ejecutan en sus propios entornos, también tienen sus propias redes virtuales. Como consecuencia, debe vincular su puerto local al "puerto virtual" de Docker (aquí 9005)
- -v : significa volumen . Esto está relacionado con el " mapeo de volumen ", que consiste en mapear directorios o archivos en su sistema local a directorios en el contenedor.
- –Config.file : el archivo de configuración que utilizará el contenedor Prometheus Docker.
- –Storage.tsdb.path : la ubicación donde Prometheus va a almacenar los datos (es decir, la serie temporal almacenada).
- –Net : queremos que el contenedor Prometheus Docker se comparta en la misma red que el host. De esta forma, podemos configurar exportadores en contenedores individuales y exponerlos al contenedor Prometheus Docker.

Para asegurarse de que todo funcione correctamente, enumere los contenedores en ejecución en su instancia.

```sh
docker container ls -a

```

Si su contenedor no se está ejecutando, aquí hay un comando para inspeccionar lo que sucedió durante la inicialización del contenedor.

```sh
docker container logs -f --since 10m <container_id>
```
Este comando es muy útil cuando sucede algo incorrecto en su contenedor.

Sin embargo, si su contenedor está activo, debería tener acceso a la interfaz de usuario web de Prometheus, que se encuentra de forma predeterminada en http://localhost:9005

![img](https://devconnected.com/wp-content/uploads/2019/08/prometheus-web-u.png)



