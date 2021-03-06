# Cassandra DataSource for Grafana 

Apache Cassandra & DataStax Enterprise Datasource for Grafana.

## Usage

[TBD]

## Development

*This part of the documentation relates only to development of the plugin and not required if you only intended to use it.*

Frontend part is implemented using *Typescript*, *WebPack*, *ESLint* and *NPM*, backend is written on *Golang* and uses *Dep* as a dependency manager. The plugin development uses docker actively and it's recommended to have at least basic understanding of docker and docker-compose.

### Installation and Build

First, clone the project. It has to be built with docker or with locally installed tools. 

#### Docker Way (Recommended)

* `docker run --rm -v ${PWD}:/opt/gcds -w /opt/gcds node:11 npm install`
* `docker run --rm -v ${PWD}:/opt/gcds -w /opt/gcds node:11 node node_modules/webpack/bin/webpack.js`
* `docker run --rm -v ${PWD}:/go/src/github.com/ha/gcp -w /go/src/github.com/ha/gcp instrumentisto/dep ensure`
* `docker run --rm -v ${PWD}:/go/src/github.com/ha/gcp -w /go/src/github.com/ha/gcp golang go build -i -o ./dist/cassandra-plugin_linux_amd64 ./backend`

#### Locally

* `npm install`
* `webpack`
* `dep ensure`
* `go build -i -o ./dist/cassandra-plugin_linux_amd64 ./backend`

### Run Grafana, Cassandra & Studio

`docker-compose up -d`

docker-compose includes three services:

- *Grafana* by itself, the plugin is mounted as a volume to `/var/lib/grafana/plugins/cassandra`. Verbose logging is enabled.
- DataStax Enterprise (Enterprise version of Apache Cassandra, will be replaced by the OSS Cassandra soon)
- DataStax Studio, web-based UI of the Cassandra to simplify development

After the startup, the datasource should be available in the list of datasources. Also, following lines should appear in grafana logs:

```
# Frontend part registered
lvl=info msg="Starting plugin search" logger=plugins
lvl=info msg="Registering plugin" logger=plugins name="Apache Cassandra"
...
# Backend part is started and running
msg="Plugins: Adding route" logger=http.server route=/public/plugins/hadesarchitect-cassandra-datasource dir=/var/lib/grafana/plugins/cassandra/dist
msg="starting plugin" logger=plugins plugin-id=hadesarchitect-cassandra-datasource path=/var/lib/grafana/plugins/cassandra/dist/cassandra-plugin_linux_amd64 args=[/var/lib/grafana/plugins/cassandra/dist/cassandra-plugin_linux_amd64]
msg="plugin started" logger=plugins plugin-id=hadesarchitect-cassandra-datasource path=/var/lib/grafana/plugins/cassandra/dist/cassandra-plugin_linux_amd64 pid=23
msg="waiting for RPC address" logger=plugins plugin-id=hadesarchitect-cassandra-datasource path=/var/lib/grafana/plugins/cassandra/dist/cassandra-plugin_linux_amd64
msg="2020-01-16T22:08:51.619Z [DEBUG] cassandra-backend-datasource: Running Cassandra backend datasource..." logger=plugins plugin-id=hadesarchitect-cassandra-datasource
msg="plugin address" logger=plugins plugin-id=hadesarchitect-cassandra-datasource address=/tmp/plugin991218850 network=unix timestamp=2020-01-16T22:08:51.622Z
msg="using plugin" logger=plugins plugin-id=hadesarchitect-cassandra-datasource version=1
```

To read the logs, use `docker-compose logs -f grafana`.

### Making Changes

#### Frontend

`watch` isn't implemented yet. After the frontend changes it should be rebuild and grafana container should be restarted.

* `docker run --rm -v ${PWD}:/opt/gcds -w /opt/gcds node:11 node node_modules/webpack/bin/webpack.js`
* `docker-compose restart grafana`

#### Backend

With any changes done to backend, the binary file should be recompiled and *grafana* should be restarted:

* `docker run --rm -v ${PWD}:/go/src/github.com/ha/gcp -w /go/src/github.com/ha/gcp golang go build -i -o ./dist/cassandra-plugin_linux_amd64 ./backend`
* `docker-compose restart grafana`
