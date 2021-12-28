# Usage

## prerequisite
in `geolocation-services/` folder, run command below to clone the required sub module
```bash
git submodule init
git submodule update
```

## 1. nominatim-docker
go to `geolocation-services/nominatim-docker` folder
```bash
cd ./nominatim-docker
```

build nominatim image
```bash
docker build -t nominatim .
```

## 2. photon-docker
go to `geolocation-services/photon-docker` folder
```bash
cd ./photon-docker
```

build photon image
```bash
docker build -t photon .
```

(if linux, replace 'host.docker.internal' in Dockerfil with the internal IP address of the Docker host). You can get the Docker internal IP from command below:
```bash
ifconfig docker0
```
NOTE: Nominatim container has to be running as photon will connect to nominatim's postgres for document reference during docker image build.


## 3. graphhopper-docker
go to `geolocation-services/graphhopper` folder
```bash
cd ./graphhopper
```
---
checkout version 0.13 by running command below
```bash
git checkout 0.13
```
---
visit https://download.geofabrik.de/index.html, browse to respective map's pbf screen to extract the URL. For example, malaysia map: https://download.geofabrik.de/asia/malaysia-singapore-brunei.html

copy the URL `asia/malaysia-singapore-brunei.html`

* replace the trailing `.html` with `.pbf`
* replace the `/` with `_`
```bash
asia_malaysia-singapore-brunei.pbf
```
* open `geolocation-services/graphhopper/Dockerfile`
* change `https://deb.nodesource.com/setup_11.x` to `https://deb.nodesource.com/setup_12.x`
* change `CMD [ "/data/europe_germany_berlin.pbf" ]` to `CMD [ "-i", "/data/asia_malaysia-singapore-brunei.pbf", "-o", "malaysia-singapore-brunei-gh" ]` from previous step
* configure jvm heapsize `ENV JAVA_OPTS "-server -Xconcurrentio -Xmx1g -Xms1g -XX:+UseG1GC -Ddw.server.applicationConnectors[0].bindHost=0.0.0.0 -Ddw.server.applicationConnectors[0].port=8989"` to larger figure depending on map size

---
open `geolocation-services/graphhopper/graphhopper.sh`, replace script block line 246 with:
```bash
if [[ "$ACTION" = "web" ]]; then
  export MAVEN_OPTS="$MAVEN_OPTS $JAVA_OPTS"
  if [[ "$RUN_BACKGROUND" == "true" ]]; then
    exec "$JAVA" $JAVA_OPTS -Dgraphhopper.datareader.file="$OSM_FILE" -Dgraphhopper.graph.location="$GRAPH" \
                 -Dgraphhopper.graph.flag_encoders=car \
                 -Dgraphhopper.graph.location=./graph-cache \
                 -Dgraphhopper.graph.bytes_for_flags=12 \
                 $GH_WEB_OPTS -jar "$JAR" server $CONFIG <&- &
    
    if [[ "$GH_PID_FILE" != "" ]]; then
       echo $! > $GH_PID_FILE
    fi
    exit $?
  else
    # TODO how to avoid duplicative command for foreground and background?
    exec "$JAVA" $JAVA_OPTS -Dgraphhopper.datareader.file="$OSM_FILE" -Dgraphhopper.graph.location="$GRAPH" \
                 -Dgraphhopper.graph.flag_encoders=car \
                 -Dgraphhopper.graph.location=./graph-cache \
                 -Dgraphhopper.graph.bytes_for_flags=12 \
                 $GH_WEB_OPTS -jar "$JAR" server $CONFIG
    # foreground => we never reach this here
  fi
```

---
build graphhopper image
```bash
docker build -t graphhopper .
```

## 4. Run container

### 4.1. Docker Run - Nominatim

start nominatim container with malaysia map
```bash
docker run -d -p 8080:8080 -p 5432:5432 \
-e NOMINATIM_PBF_URL='https://download.geofabrik.de/asia/malaysia-singapore-brunei-latest.osm.pbf' \
--name nominatim -ti nominatim
```

[optional] map to different host port
```bash
docker run -d -p 8181:8080 -p 5432:5432 \
-e NOMINATIM_PBF_URL='https://download.geofabrik.de/asia/malaysia-singapore-brunei-latest.osm.pbf' \
--name nominatim -ti nominatim
```

test nominatim container
```bash
http://localhost:8080/
```
* change `NOMINATIM_PBF_URL` to other pbf's path for other map, e.g. -> `https://download.geofabrik.de/asia/japan-latest.osm.pbf`
* this might take hours / days to import if map being imported is too big 


### 4.2. Docker Run - Photon

start photon container link to nominatim container's db
```bash
docker run -p 2322:2322 --name photon --link nominatim:nominatim -ti photon
```


test photon container
```bash
curl "http://localhost:2322/api?q=klcc"
```


### 4.3. Docker Run - GraphHopper
start graphhopper container
```bash
docker run -d --name graphhopper -v <path_geolocation-services_directory>/data:/data -p 8989:8989 graphhopper
```

* this might take hours to load the target pbf, depending on the file size


### 4.4. Docker-compose - ALL
browse to `geolocation-services/` and update the `docker-compose.yml` accordingly to point to correct images / ports / volume.

run command
```bash
docker-compose up
```

