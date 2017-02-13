# docker-compose-nba
Compose Repostitory of NBA cluster


In this repository are some Docker Compose files for running a NBA cluster in docker containers. This will install `api`, `etl` , `elasticsearch`, `kibana`.

You can use `docker-compose-with-minio.yaml` to install it with Minio S3 service (https://minio.io)

You can use `docker-compose-with-grafana.yml` to install it with Grafana (https://grafana.net/)

You can use `docker-compose-all.yaml` to add Minio and Grafana.

### Quickstart
To start a cluster without configuring run
```
docker-compose -f docker-compose-all.yaml -p my-cluster up -d
```
To check your running instances
```
docker ps
```
To show logs of a container
```
docker logs <container name>
```
To stop the cluster
```
docker-compose -f docker-compose-all.yaml -p my-cluster down
```
Where `-f` links to your docker compose yaml and `-p` is your cluster name and `-d` is to run in the background.

The api will be available on port 8080, kibana on 5601. To run the import run
```
docker exec my-cluster_etl_1 sh/import-all
```
Where `my-cluster` this the cluster name and `sh/import-all` is the command you want to run. Run
`docker exec my-cluster_etl_1 ls sh/` to show all available commands.

### Configuring your cluster
You can edit the yaml file. When after you made changes you should run this command again
```
docker-compose -f docker-compose-all.yaml -p my-cluster up -d
```

You should think about the following values.
```
atzedevries/docker-nba-api:2.05-44-g84ab21e3
```
This is the build of the api. You can find the avaiable versions at
https://hub.docker.com/r/atzedevries/docker-nba-api/tags/. The tag (after ':' ) is made up of
The most recent tag (`2.0.5`), the number of commits after this tag (`44`) and the shorthash of the commit (`84ab21e3`). You can find the commit messages until this commit on github via https://github.com/naturalis/naturalis_data_api/commits/<shorthash>. In this example it would be https://github.com/naturalis/naturalis_data_api/commits/84ab21e3

```
ES_JAVA_OPTS: "-Xms512m -Xmx512m"
```
This is the amount of memory elasticsearch gets. It should be half of system memory. Both the values of `-Xms` and `-Xms` should be te same. For a 8GB machine it should be `ES_JAVA_OPTS: "-Xms4g -Xmx4g"`

```
- /es-data:/usr/share/elasticsearch/data
```
This is where your elasticseach data should sit. In this case it is `/es-data`.

```
image: atzedevries/docker-nba-etl:2.05-53-g08338884
```
The version of the ETL module. Available tags are found here https://hub.docker.com/r/atzedevries/docker-nba-api/tags/

```
- /data/nba-import/data:/payload/data
- /data/nba-import/log:/payload/software/log
```
These are the paths where your import data sits and where you want to put your log files. In this case `/data/nba-import/data` should contain the directoryes `crs`, `col` etc. `/data/nba-import/log` Should be a directory on your system.

#### Extra products
If you choose to use `docker-compose-all.yaml` then minio and grafana is also installed. For minio you can configure
```yaml
environment:
  # minimal 5 chars
  MINIO_ACCESS_KEY: piets
  # minimal 8 chars
  MINIO_SECRET_KEY: klaasklaas
volumes:
  - /data/snapshots:/export
```

Where `MINIO_ACCESS_KEY` is your accesskey (ala username) and `MINIO_SECRET_KEY` is your secretkey (password). `/data/snapshots` is the direcory where minio stores your data.

Minio is a S3 like service, you can use this in combination with ES to take and restore snapshots fully via http. To add the repositiory should first find the minio ip.
```
docker inspect <your cluster name>_minio_1 -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```
Minio is reachable on port 9000

Grafana is a tool like kibana. You can configure
```
GF_SECURITY_ADMIN_PASSWORD: jan
```
This is the password for gafana. Grafana is reachable on port 3000
