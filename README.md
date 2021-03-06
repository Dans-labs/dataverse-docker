## DataverseEU Docker module
Dataverse Docker module was developed by [DANS](http://dans.knaw.nl) (Data Archiving and Networked Services) to run Dataverse data repository on Kubernetes and other Cloud services supporting Docker.

### Installation

##### Prerequisites
First step is download of required software for authomatic Dataverse installation (PostgreSQL, SOLR, Glassfish, Dataverse)
`./initial.bash`

##### Using Docker
* Make sure you have docker and docker-compose installed
* Build all containers with Docker Compose `docker-compose build` 
* Run `docker-compose up` so start Dataverse

Standalone Dataverse in English should be running on http://localhost:8085

If it's not coming up please check if all required containers are up: `docker ps`

```

CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                          NAMES

3a30792b22fe        dockereu_dataverse                    "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:440->443/tcp, 0.0.0.0:8085->8080/tcp   dataverse

8903ffab7d79        dockereu_solr                         "/entrypoint.sh solr"    About a minute ago   Up About a minute   0.0.0.0:8985->8983/tcp                         solr

e652e204e6bb        dockereu_postgres                     "docker-entrypoint.sh"   14 minutes ago       Up About a minute   0.0.0.0:5435->5432/tcp                         db
```

#### Multilingual support
If you want to start multilingual web interface please run

`docker-compose -f ./docker-multilingual.yml up`

After 20-25 minutes or so you’ll get localized Dataverses running on localhost:8085, localhost:8086 etc (see specification in .yaml file)

Check again if Dataverse containers are running by `docker ps`

```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                          NAMES

cc84feb9760c        dockereu_dataverse_fr                 "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:446->443/tcp, 0.0.0.0:8088->8080/tcp   dataverse_fr

56229df080d9        dockereu_dataverse_es                 "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:450->443/tcp, 0.0.0.0:8090->8080/tcp   dataverse_es

14d7719ea79e        dockereu_dataverse_de                 "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:447->443/tcp, 0.0.0.0:8086->8080/tcp   dataverse_de

4af3942245ee        dockereu_dataverse_ua                 "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:453->443/tcp, 0.0.0.0:8089->8080/tcp   dataverse_ua

3a30792b22fe        dockereu_dataverse                    "/opt/dv/entrypoint.b"   About a minute ago   Up About a minute   0.0.0.0:440->443/tcp, 0.0.0.0:8085->8080/tcp   dataverse

8903ffab7d79        dockereu_solr                         "/entrypoint.sh solr"    About a minute ago   Up About a minute   0.0.0.0:8985->8983/tcp                         solr

e652e204e6bb        dockereu_postgres                     "docker-entrypoint.sh"   14 minutes ago       Up About a minute   0.0.0.0:5435->5432/tcp                         db
```

##### Specific language selection
If you want to run specific version of Dataverse then start containers separately, for example, for French
`docker-compose -f ./docker-multilingual.yml up dataverse_fr`

#### Going from Docker Compose to Kubernetes
If you want to run Dataverse on Kubernetes there is convertor called [Kompose] (https://github.com/kubernetes/kompose)

Install it and run the command:
`kompose convert -f ./docker-multilingual.yml` 

You should see Kubernetes specifications created:

```
INFO Container name in service "dataverse_de" has been changed from "dataverse_de" to "dataverse-de" 
INFO Service name in docker-compose has been changed from "dataverse_de" to "dataverse-de" 
INFO Container name in service "dataverse_es" has been changed from "dataverse_es" to "dataverse-es" 
INFO Service name in docker-compose has been changed from "dataverse_es" to "dataverse-es" 
INFO Container name in service "dataverse_fr" has been changed from "dataverse_fr" to "dataverse-fr" 
INFO Service name in docker-compose has been changed from "dataverse_fr" to "dataverse-fr" 
INFO Container name in service "dataverse_ua" has been changed from "dataverse_ua" to "dataverse-ua" 
INFO Service name in docker-compose has been changed from "dataverse_ua" to "dataverse-ua" 
INFO Kubernetes file "dataverse-service.yaml" created 
INFO Kubernetes file "dataverse-de-service.yaml" created 
INFO Kubernetes file "dataverse-es-service.yaml" created 
INFO Kubernetes file "dataverse-fr-service.yaml" created 
INFO Kubernetes file "dataverse-ua-service.yaml" created 
INFO Kubernetes file "postgres-service.yaml" created 
INFO Kubernetes file "solr-service.yaml" created  
INFO Kubernetes file "dataverse-deployment.yaml" created 
INFO Kubernetes file "dataverse-de-deployment.yaml" created 
INFO Kubernetes file "dataverse-es-deployment.yaml" created 
INFO Kubernetes file "dataverse-fr-deployment.yaml" created 
INFO Kubernetes file "dataverse-ua-deployment.yaml" created 
INFO Kubernetes file "postgres-deployment.yaml" created 
INFO Kubernetes file "postgres-claim0-persistentvolumeclaim.yaml" created 
INFO Kubernetes file "solr-deployment.yaml" created 
INFO Kubernetes file "solr-claim0-persistentvolumeclaim.yaml" created
``` 

Now you can start all created services with kubectl, for example: `kubectl create -f dataverse-deployment.yaml`

#### Deployment using Dataverse image from Docker Hub

* create image out of the installed container with removed temporary files after Dataverse will go up on port 8085:
`docker commit dataversedocker_dataverse`
* Find ID of new image with running (it will show you newly created image on the top of the list):
`docker images`
* Archive this image with command like (for example, imageID is 33c86dbfdc9e):
`docker save 33c86dbfdc9e > dataverse.tar`
* Push this image to Docker Hub, "username" should be your login there:
```
export DOCKER_ID_USER="username"
docker login
docker tag dataversedocker_dataverse $DOCKER_ID_USER/dataverse
docker push $DOCKER_ID_USER/dataverse
```

#### Warning

If not all languages are coming up in the same time please increase RAM for Docker (not less than 10Gb for 5 languages). 

#### To Do

Health check support should be added to get Dataverse installation process from Docker more sustainable.

