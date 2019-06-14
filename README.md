# Docker Compose Introduction
This project contains example projects using Docker Compose.  To use these projects, clone the repository with ```git clone https://github.com/danial-k/docker-compose-intro.git``` and in each of the sub-directories run ```docker-compose up```.  If your machine does not already have docker-compose installed, follow the [official installation instructions](https://docs.docker.com/compose/install/).
This will create a new isolated network and bring up the containers specified in the ```docker-compose.yaml``` files in that network.  To remove the containers and the isolated network, run ```docker-compose down```.

This project contains the following examples:
- [nginx reverse proxy for load balancing](nginx-load-balancer)
- [Drools Business Central UI and KIE execution server for business rules automation](drools-workbench-kie-server)
