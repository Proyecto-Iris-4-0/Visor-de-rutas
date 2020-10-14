# Visor de rutas
En este repositorio se encuentra un código Python para la visualización con JupyterNotebook de rutas generadas por distintas APIs de mapas.

Las API's empleadas han sido Google, Graphhopoper y OSRM. Para que el código funcione se necesita disponer de una API key de Google y tener instaladas Graphhopper y OSRM.

## Instalación de Graphhopper mediante Docker.

Clonar el siguiente repositorio: 
```
$ git clone https://github.com/graphhopper/graphhopper
```

Crea una carpeta "data" para almacenar los mapas:
```
$ makdir ~/data
```

Descarga la última versión del mapa de España desde el repositorio openstreetmap.fr:
```
$ wget http://download.openstreetmap.fr/extracts/europe/spain-latest.osm.pbf
```
Ahora hay que editar el Dockerfile del repositorio Graphhopper para que reconozca el nuevo mapa. Reemplaza la última línea por:
```
CMD [ "/data/spain-latest.osm.pbf" ]
```
Ahora en el mismo Dockerfile, hay que cambiar la variable de entorno "JAVA_OPTS" para darle más memoria a la JVM. Con 4 GB es suficiente. Hay que reemplazar la línea:

```
ENV JAVA_OPTS "-server -Xconcurrentio -Xmx1g -Xms1g -XX:+UseG1GC -Ddw.server.application_connectors[0].bind_host=0.0.0.0 -Ddw.server.application_connectors[0].port=8989"
```
 por:
```
ENV JAVA_OPTS "-server -Xconcurrentio -Xmx4g -Xms4g -XX:+UseG1GC -Ddw.server.application_connectors[0].bind_host=0.0.0.0 -Ddw.server.application_connectors[0].port=8989"
```

Ahora ya estamos en disposición de construir la imagen del Docker:

```
$ docker build -t graphhopper:master .  
```
Es posible que salga algún warning.

Y levantar el contenedor:
```
$ docker run -d --name graphhopper -v /home/$USER/data:/data -p 8989:8989 graphhopper:master
```
En el puerto 8989 se debería haber levantado ya el servicio.


## Instalación de OSRM mediante Docker.

Usaremos el mismo mapa que hemos descargado para el caso anterior y que hemos dejado en la carpeta ~/data. Hay que "correr" las siguientes imágenes de Docker:
```
$ docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/spain-latest.osm.pbf
```
Ahora se lanzan dos contenedores que van calculando y generando las matrices de distancias:
```
$ docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/spain-latest.osrm
```
```
$ docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/spain-latest.osrm
```

Ahora lanzamos el servidor en el puerto 5000
```
$ docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/spain-latest.osrm
```



