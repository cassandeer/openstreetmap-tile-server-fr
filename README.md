# openstreetmap-tile-server in 🇫🇷

This project allows you to easily set up an OpenStreetMap PNG tile server given a .osm.pbf file with French label for location.

Inspired by [osmfr-cartocss](https://github.com/cquest/osmfr-cartocss) for mapnik file.

## Building your image

```
docker build -f Dockerfile . -t my-french-openstreetmap-tile-server
```

## Setting up the server

First create a Docker volume to hold the PostgreSQL database that will contain the OpenStreetMap data:

    docker volume create osm-data

Next, download an `.osm.pbf` extract from [geofabrik.de](https://download.geofabrik.de/) for the region that you're interested in. You can then start importing it into PostgreSQL by running a container and mounting the file as `/data/region.osm.pbf`. For example:

```
docker run \
    -v /absolute/path/to/<the-country-you-want>.osm.pbf:/data/region.osm.pbf \
    -v osm-data:/data/database/ \
    my-french-openstreetmap-tile-server \
    import
```

If the container exits without errors, then your data has been successfully imported and you are now ready to run the tile server.

Note that the import process requires an internet connection. The run process does not require an internet connection. If you want to run the openstreetmap-tile server on a computer that is isolated, you must first import on an internet connected computer, export the `osm-data` volume as a tarfile, and then restore the data volume on the target computer system.

Also when running on an isolated system, the default `index.html` from the container will not work, as it requires access to the web for the leaflet packages.

## Running the server

Run the server like this:

```
docker run \
    -p 8080:80 \
    -v osm-data:/data/database/ \
    -d my-french-openstreetmap-tile-server \
    run
```

Your tiles will now be available at `http://localhost:8080/tile/{z}/{x}/{y}.png`. The demo map in `leaflet-demo.html` will then be available on `http://localhost:8080`. Note that it will initially take quite a bit of time to render the larger tiles for the first time.

### Using Docker Compose

The `docker-compose.yml` file included with this repository shows how the aforementioned command can be used with Docker Compose to run your server.

### Preserving rendered tiles

Tiles that have already been rendered will be stored in `/data/tiles/`. To make sure that this data survives container restarts, you should create another volume for it:

```
docker volume create osm-tiles
docker run \
    -p 8080:80 \
    -v osm-data:/data/database/ \
    -v osm-tiles:/data/tiles/ \
    -d overv/openstreetmap-tile-server \
    run
```

**If you do this, then make sure to also run the import with the `osm-tiles` volume to make sure that caching works properly across updates!**


## License

```
Copyright 2019 Alexander Overvoorde

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
