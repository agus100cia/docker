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
 
docker run \
--name prometheus \
-d \
-p 9005:9090 \
--user 991:977 \
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


Para asegurarse de que todo funcione correctamente, enumere los contenedores en ejecución en su instancia.

```sh
docker container ls -a

```

```sh
[naadbd01@quisrvbigdata9 prometheus]$ docker container ls -a
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS                      PORTS                                                                                                                                           NAMES
99434df1735f        prom/prometheus         "/bin/prometheus --c…"   4 seconds ago       Up 3 seconds                        
```


Si su contenedor no se está ejecutando, aquí hay un comando para inspeccionar lo que sucedió durante la inicialización del contenedor.

```sh
docker container logs -f --since 10m <container_id>
```
Este comando es muy útil cuando sucede algo incorrecto en su contenedor.

Sin embargo, si su contenedor está activo, debería tener acceso a la interfaz de usuario web de Prometheus, que se encuentra de forma predeterminada en http://localhost:9005

![img](https://devconnected.com/wp-content/uploads/2019/08/prometheus-web-u.png)


Sin embargo, Prometheus solo no es muy útil. Como consecuencia, vamos a instalar el exportador de Node : un exportador responsable de recopilar y agregar métricas relacionadas con el rendimiento del sistema Linux.

## Export node

El exportador de Node también viene con su propia imagen de Docker.

De manera similar a lo que hicimos con la imagen de Prometheus Docker, creemos un usuario para Node Exporter.

```sh
docker pull prom/node-exporter
sudo mkdir -p /home/naadbd01/docker/node-exporter
sudo useradd -rs /bin/false node_exporter
sudo chown -R node_exporter:node_exporter /home/naadbd01/docker/node-exporter
cat /etc/passwd | grep node_exporter
node_exporter:x:990:976::/home/node_exporter:/bin/false

```

Vamos a ejecutar el contenedor

```sh
docker run \
--name node-exporter \
-d \
-p 9004:9100 \
--user 990:976 \
-v "/home/naadbd01/docker/node-exporter:/hostfs" \
prom/node-exporter \
--path.rootfs=/hostfs
```


Verifique que node-exporter este funcionando

```sh
curl http://localhost:9006/metrics

node_scrape_collector_success{collector="arp"} 1
node_scrape_collector_success{collector="bcache"} 1
node_scrape_collector_success{collector="bonding"} 1
node_scrape_collector_success{collector="conntrack"} 1
node_scrape_collector_success{collector="cpu"} 1
```

Ahora necesitamos vincular a Prometheus con el exportador de nodos

## Vincular Prometheus con node-exporter

Vamos a modificar el archivo de configuracion de prometheus. La idea es agregar un job_name que apunte a node-exporter

```yml
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

  - job_name: 'node-exporter'
    scrape_interval: 10s
    static_configs:
    - targets: ['localhost:9004']

```

No es necesario reiniciar el contenedor Prometheus Docker para que se apliquen las modificaciones.

Para reiniciar su configuración de Prometheus, simplemente envíe una señal SIGHUP a su proceso de Prometheus.

Primero, identifique el PID de su servidor Prometheus.

```sh
ps aux | grep prometheus

prometh+ 54459  0.2  0.0 906312 38492 ?        Ssl  18:44   0:04 /bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheu
naadbd01 55157  0.0  0.0 112652   968 pts/43   S+   19:09   0:00 grep --color=auto prometheus

```

Para el caso el PID es 54459. Envíe una señal SIGHUP a este proceso para que se reinicie la configuración.

```sh
sudo kill -HUP 54459
 
 ```
 
 Ahora, su instancia de Prometheus debe tener el exportador de nodos como destino. Para verificarlo, diríjase a 
 
 http://localhost:9005 y verifique que sea el caso.
 
 ## Grafana
 
 Ahora necesitamos de un visualizador para graficar las metricas y para eso vamos a utilizar grafana
 
 ```sh
 docker pull grafana/grafana
 sudo mkdir -p /home/naadbd01/docker/grafana
 
 ```
 
 Ejecutamos el contenedor
 
```sh
docker run \
--name grafana \
-d \
-p 9006:3000 \
grafana/grafana 
 
```

Por defecto el usuario y clave es admin/admin

![img](https://devconnected.com/wp-content/uploads/2019/08/grafana-default.png)

De forma predeterminada, se le redirigirá a la página de inicio. Haga clic en la opción "Agregar una fuente de datos".

![img](https://devconnected.com/wp-content/uploads/2019/08/home.png)

Elija una fuente de datos de Prometheus y haga clic en " Seleccionar "

![img](https://devconnected.com/wp-content/uploads/2019/08/prometheus-datasource-1.png)

Aquí está la configuración para su host Prometheus.

![img](https://devconnected.com/wp-content/uploads/2019/08/datasource-prometheus.png)

![img](https://devconnected.com/wp-content/uploads/2019/08/save-test.png)

Como el exportador de nodo es un exportador bastante popular, hay desarrolladores que ya crearon paneles de control para él.

Como consecuencia, vamos a instalar el panel de Exportador de Nodos en Prometheus.

En el menú de la izquierda, haga clic en el icono "Más", luego en "Importar".

![img](https://devconnected.com/wp-content/uploads/2019/08/import.png)

En el cuadro de texto del panel de Grafana, escriba 1860 y espere a que Grafana recupere la información del panel automáticamente.

![img](https://devconnected.com/wp-content/uploads/2019/08/dashboard-import.png)

En la siguiente ventana, seleccione Prometheus como fuente de datos y haga clic en " Importar ".

![img](https://devconnected.com/wp-content/uploads/2019/08/import-2.png)

¡Eso es! Su panel ahora debería crearse automáticamente.

![img](https://devconnected.com/wp-content/uploads/2019/08/final-dash.png)



## Prometheus con nginx

nginx en sí ya viene con un punto final de estado por sí solo, que se puede habilitar usando ngx_http_stub_status_module . Para hacer esto, tenemos que abrir nuestro nginx.conf y agregar una ubicación separada:

```yml
http {
    index   index.html;
    server {
        location /metrics {
            stub_status on;
        }
    }
}
```

Para ejecutar el cambio en el contenedor sin reiniciarlo puede usar:

```ssh
docker exec <nginxcontainername/id> nginx -s reload
```

Usando esta configuración, tendremos un endpoint /metrics que nos proporcionará alguna información, por ejemplo:

```html
Active connections: 3 
server accepts handled requests
 3100 3100 191419 
Reading: 0 Writing: 1 Waiting: 2 

```

### Usando el exportador de prometheus

Si bien ya es genial que tengamos algunas métricas ahora, no están listas para ser consumidas por Prometheus. Para hacer eso, tenemos que crear un punto final legible por Prometheus que proporcione estas métricas.

Una de las herramientas que podemos usar para hacer esto es nginx-prometheus-exporter , que se puede ejecutar usando Docker, por ejemplo:

```ssh
docker pull nginx/nginx-prometheus-exporter

```

Para ejecutarlo:

```ssh
docker run \
--name nginx-prometheus-exporter \
-d \
-p 9113:9113 nginx/nginx-prometheus-exporter:0.2.0 \
-nginx.scrape-uri=http://quisrvbigdata9:8800/metrics
  
```


