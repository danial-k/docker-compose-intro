# nginx load balancer Docker Compose example
In this example, an nginx web server container wil sit in front of three basic containers.  The web server will act as a reverse proxy, alterating requests between the three back-end servers sequentially (round-robin, see [nginx manual](https://nginx.org/en/docs/http/load_balancing.html) for more configuration options). 

Clone this repository with ```git clone https://github.com/danial-k/docker-compose-intro.git``` and open ```docker-compose-intro/nginx-load-balancer``` in an IDE.  Examine the contents of ```docker-compose.yaml``` and ```nginx/default/conf``` (the web server config file).

```shell
cd nginx-load-balancer
docker-compose up
```
If your machine does not already have docker-compose installed, follow the [official installation instructions](https://docs.docker.com/compose/install/).  This command should bring up the three backend services up first then the nginx reverse proxy.  The dependency is set by the ```depends_on``` directive in the [docker-compose.yaml](docker-compose.yaml) file.

Repeatedly visiting http://127.0.0.1:3030/whoami should alternate between the hostnames in a round-robin manner.
