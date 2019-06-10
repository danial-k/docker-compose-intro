# nginx load balancer Docker Compose example
In this example, an nginx web server container sits in front of three containers that will echo their hostname.  The web server will act as a reverse proxy, alternating requests between the three back-end servers sequentially (round-robin, see [nginx manual](https://nginx.org/en/docs/http/load_balancing.html) for more load-balancing strategies). 

Clone this repository with ```git clone https://github.com/danial-k/docker-compose-intro.git``` and open ```docker-compose-intro/nginx-load-balancer``` in an IDE.  Examine the contents of ```docker-compose.yaml``` and ```nginx/default/conf``` (the web server config file).

```shell
cd nginx-load-balancer
docker-compose up
```
If your machine does not already have docker-compose installed, follow the [official installation instructions](https://docs.docker.com/compose/install/).  This command should bring up the three backend services up first then the nginx reverse proxy.  The dependency is set by the ```depends_on``` directive in the [docker-compose.yaml](docker-compose.yaml) file.

Repeatedly visiting http://127.0.0.1:3030/ should alternate between the hostnames in a round-robin manner.

## What did Compose do for us?
In the background, Docker Compose:
- Creates a new isolated network using the base folder name
- Creates the containers using the directives in ```docker-compose``` in the order determined by ```depends_on```
- Captures stdout from all containers into a single tty
- Enables stopping/removing containers as a group

This is equivalent to:

```shell
docker network create nginx-load-balancer
```
Followed by creating the containers within the new network:
```shell
docker run \
--network nginx-load-balancer \
--name whoami1 \
jwilder/whoami:latest
```

```shell
docker run \
--network nginx-load-balancer \
--name whoami2 \
jwilder/whoami:latest
```

```shell
docker run \
--network nginx-load-balancer \
--name whoami3 \
jwilder/whoami:latest
```

```shell
docker run \
--network nginx-load-balancer \
--name nginx \
--mount type=bind,src=`pwd`/nginx/default.conf,dst=/etc/nginx/conf.d/default.conf \
-p 3031:80 \
nginx:1.17.0
```

Repeatedly visiting http://127.0.0.1:3031/ should alternate between the hostnames in a round-robin manner.
