# Usage

## 1. nominatim-docker
go to `geolocation-services/nominatim-docker` folder
```bash
cd ./nominatim-docker
```

build nominatim image
```bash
docker build -t nominatim .
```

start nominatim container with malaysia map
```bash
docker run -d -p 8080:8080 -p 5432:5432 \
-e NOMINATIM_PBF_URL='https://download.geofabrik.de/asia/malaysia-singapore-brunei-latest.osm.pbf' \
--name nominatim -ti nominatim
```

test nominatim container
```bash
http://localhost:8080/
```
* change `NOMINATIM_PBF_URL` to other pbf's path for other map, e.g. -> `https://download.geofabrik.de/asia/japan-latest.osm.pbf`
* this might take hours / days to import if map being imported is too big 


## 2. photon-docker
go to `geolocation-services/photon-docker` folder
```bash
cd ./photon-docker
```

build photon image
```bash
docker build -t photon .
```

start photon container link to nominatim container's db
```bash
docker run -p 2322:2322 --name photon --link nominatim:nominatim -ti photon
```

test photon container
```bash
curl "http://localhost:2322/api?q=klcc"
```

## 3. graphhopper-docker
go to `geolocation-services/` folder, run command below to clone graphhopper repo
```bash
git clone https://github.com/graphhopper/graphhopper.git
```
---
go to `geolocation-services/graphhopper` folder
```bash
cd ./graphhopper
```

---
visit https://download.geofabrik.de/index.html, browse to respective map's pbf screen to extract the URL. For example, malaysia map: https://download.geofabrik.de/asia/malaysia-singapore-brunei.html

extract from the URL `asia/malaysia-singapore-brunei.html`

* replace the trailing `.html` with `.pbf`
* replace the `/` with `_`
```bash
asia_malaysia-singapore-brunei.pbf
```
* open `geolocation-services/graphhopper/Dockerfile`
* change `CMD [ "/data/europe_germany_berlin.pbf" ]` to `CMD [ "-i", "/data/asia_malaysia-singapore-brunei.pbf", "-o", "malaysia-singapore-brunei-gh" ]` from previous step
* configure jvm heapsize `ENV JAVA_OPTS "-server -Xconcurrentio -Xmx1g -Xms1g -XX:+UseG1GC -Ddw.server.applicationConnectors[0].bindHost=0.0.0.0 -Ddw.server.applicationConnectors[0].port=8989"` to larger figure depending on map size

---

build graphhopper image
```bash
docker build -t graphhopper .
```

start graphhopper container 
```bash
docker run -d --name graphhopper -v <path_geolocation-services_directory>/data:/data -p 8989:8989 graphhopper
```

* this might take hours to load the target pbf, depending on the file size