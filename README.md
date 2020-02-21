# Usage

## 1. nominatim-docker
go to `geolocation/nominatim-docker` folder
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
* change to other pbf's path for other map, e.g. -> `https://download.geofabrik.de/asia/japan-latest.osm.pbf`
* this might take hours / days to import if map being imported is too big 


## 2. photon-docker
go to `geolocation/photon-docker` folder
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
go to `geolocation/` folder, run command below to clone graphhopper repo
```bash
git clone https://github.com/graphhopper/graphhopper.git
```
---
* download target pbf files from https://download.geofabrik.de/index.html
* place the files in the data folder

---
go to `geolocation/graphhopper` folder
```bash
cd ./graphhopper
```

edit `geolocation/graphhopper/Dockerfile`, change `CMD [ "/data/europe_germany_berlin.pbf" ]` to `CMD [ "/data/target_file.pbf" ]` from previous step

---

build graphhopper image
```bash
docker build -t graphhopper .
```

start graphhopper container 
```bash
docker run -d --name graphhopper -v <path_geolocation-services_directory>/data:/data -p 8989:8989 graphhopper
```