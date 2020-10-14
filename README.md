# Visor-de-rutas
En este repositorio se encuentra un código Python para la visualización con JupyterNotebook de rutas generadas por distintas APIs de mapas.

Las API's empleadas han sido Google, Graphhopoper y OSRM. Para que el código funcione se necesita disponer de una API key de Google y tener instaladas Graphhopper y OSRM.

## Instalación de Graphhopper mediante Docker.

Clonar el siguiente repositorio: 
$ https://github.com/graphhopper/graphhopper

Crea una carpeta "data" para almacenar los mapas:

$ makdir ~/data

Descarga la última versión del mapa de España desde el repositorio openstreetmap.fr:
$ wget http://download.openstreetmap.fr/extracts/europe/spain-latest.osm.pbf

Ahora hay que editar el Dockerfile del repositorio Graphhopper para que reconozca el nuevo mapa. Reemplaza la última línea por:

CMD [ "/data/spain-latest.osm.pbf" ]

Ahora en el mismo Dockerfile, hay que cambiar la variable de entorno "JAVA_OPTS" para darle más memoria a la JVM. Con 4 GB es suficiente. Hay que reemplazar la línea:

ENV JAVA_OPTS "-server -Xconcurrentio -Xmx1g -Xms1g -XX:+UseG1GC -Ddw.server.application_connectors[0].bind_host=0.0.0.0 -Ddw.server.application_connectors[0].port=8989"

 por:

ENV JAVA_OPTS "-server -Xconcurrentio -Xmx4g -Xms4g -XX:+UseG1GC -Ddw.server.application_connectors[0].bind_host=0.0.0.0 -Ddw.server.application_connectors[0].port=8989"


Ahora ya estamos en disposición de construir la imagen del Docker:

$ docker build -t graphhopper:master .  

Es posible que salga algún warning.

Y levantar el contenedor:

$ docker run -d --name graphhopper -v /home/$USER/data:/data -p 8989:8989 graphhopper:master

En el puerto 8989 se debería haber levantado ya el servicio.

