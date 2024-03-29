# Drools Workbench KIE Execution Server Docker Compose example
[Drools](https://docs.jboss.org/drools/release/7.18.0.Final/drools-docs/html_single/) is an open source business rules management system (BRMS) designed to offload complex decision making from applications into a centrally managed rules repository. The suite includes Business Central, a web-based UI for authoring and managing rules and KIE execution servers for providing REST endpoints to evaluate rules.
In this example, two containers will be used:
- a Business Central Workbench controller Docker container used to manage the project and execution servers
- a KIE Execution Server Docker container to have rules pushed to it from Business Central and provide an API for executing rules

## Starting with Docker Compose
Clone this repository with ```git clone https://github.com/danial-k/docker-compose-intro.git``` and open the ```
drools-workbench-kie-server``` directory in an IDE. Examine the contents of [docker-compose.yaml](docker-compose.yaml).  Note that credentials are hard coded for demonstration purposes only and should be changed to environment variables or another means of moving credentials out of source control. Also examine the contents of [wait-for-workbench.sh](wait-for-workbench.sh).  This script polls the Business Central API to check for availability as the Docker engine [cannot know](https://docs.docker.com/compose/startup-order/) when Business Central is actually ready to receive connections.  To start the containers with Docker Compose, run:
```shell
docker-compose up
```
If your machine does not already have docker-compose installed, follow the [official installation instructions](https://docs.docker.com/compose/install/). This command should bring up a KIE execution server first then the Business Central UI. The dependency is set by the ```depends_on directive``` in the docker-compose.yaml file but it is important to note that Docker cannot know when Business Central is actually ready to receive connections, even though from Docker's perspective the container is 'up'.  For this reason, a custom script ([wait-for-workbench.sh](wait-for-workbench.sh)) is required to keep polling the container for our definition of 'up'.

Following startup, the Business Central workbench should be available at http://127.0.0.1:3035/business-central, with ```admin``` and ```admin``` as the credentials.  The Controller API documentation is available at http://127.0.0.1:3035/business-central/docs.  Note that additional API endpoints for other operations are documented [here](https://docs.jboss.org/drools/release/7.18.0.Final/drools-docs/html_single/#knowledge-store-rest-api-endpoints-ref_decision-tables).  In the Business Central workbench, visiting ```Menu``` > ```Deploy``` > ```Execution Servers``` should also show the remote KIE execution server.  The KIE execution server documentation should be available at http://127.0.0.1:3036/kie-server/docs.

## Importing project into Business Central
Navigate to the projects page and select Import Project. In the Repostitory URL, enter https://github.com/danial-k/drools-sample.git then select import. Note that it is also possible to import from the local file system with ```file:///<path>```, for example, if a project was mounted as a Docker volume. This will instruct the importer to read from the local container file system and import the project into its own local git repository. Select the project and choose ```OK``` to being the import process. Click ```Deploy``` in the top right to push the rule package to all availabile execution servers.  To view the project in the local Maven repository, navigate to ```Settings``` > ```Artifacts```.  At this point, the API should be able to process requests 

## Making API requests
If using Postman, create a new collection and set basic auth options to admin and admins so requests will inherit authentication configuration information.

Execute runtime commands
- Type: ```POST```
- Authorization: basic (admin:admin)
- URL: http://127.0.0.1:3036/kie-server/services/rest/server/containers/instances/app_1.0-SNAPSHOT
- Body type: ```raw```
- Content type: ```application/json```
- Headers: ```Accept```:```application/json```
- Body:
```
{
    "commands": [
        {
            "insert": {
                "object": {
                    "com.example.sample.Message": {
                        "type": "Hello"
                    }
                },
                "out-identifier": "Message"
            }
        },
        {
            "fire-all-rules": {
                "max": -1,
                "out-identifier": "firedActivations"
            }
        }
    ]
}
```

# What did Compose do for us?
In the background, Docker Compose:
- Created a new isolated network using the base folder name
- Creates the containers using the directives in docker-compose in the order determined by depends_on
- Captured stdout from all containers into a single tty
- Enabled stopping/removing containers as a group

This is equivalent to creating a network:
```shell
docker network create drools-workbench-kie-server
```

Followed by starting a Business Central UI Docker container:
```shell
docker run \
-p 3040:8080 \
--name drools-workbench \
--hostname drools-workbench \
--network drools-workbench-kie-server \
jboss/drools-workbench-showcase:7.18.0.Final
```

Waiting for Business Central to become available, then starting up a KIE execution server container.  To view a list of execution server instances, (currenly none) visit http://127.0.0.1:3040/business-central/rest/controller/management/servers.   To start a new KIE execution server:
```shell
docker run \
-p 3041:8080 \
--name kie-server \
--hostname kie-server \
--network drools-workbench-kie-server \
--env KIE_SERVER_CONTROLLER=http://drools-workbench:8080/business-central/rest/controller \
--env KIE_SERVER_LOCATION=http://kie-server:8080/kie-server/services/rest/server \
--env KIE_SERVER_USER=admin \
--env KIE_SERVER_PWD=admin \
--env KIE_MAVEN_REPO=http://drools-workbench:8080/business-central/maven2 \
jboss/kie-server-showcase:7.18.0.Final
```
```KIE_SERVER_CONTROLLER``` is the path to the Business Central remote controller. ```KIE_SERVER_LOCATION``` is a self-identifying callback URL given to Business Central when it needs to make API calls back to this KIE container.```KIE_MAVEN_REPO``` is the remote Maven repository on Business Central used to retrieve built artifacts.

The endpoints will be available per the previous Docker Compose section with different ports.
