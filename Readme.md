# Docker Basics 🐳

### What is Docker ?

Docker is a platform as a service (PaaS) that provides us the instances of the services which we need to run on our system without getting into too much trouble of getting latest versions and all but to just simply do a "docker run" 🚀

### Docker Commands :

- Run (start a container)
```bash
docker run <image_name>

# Eg: docker run nginx
```

- PS (list all the running containers)
```bash
# To get all the container currently running
docker ps

# To get all the containers currently and previously running
docker ps -a
```

- Stop (to stop the docker container)
```bash
docker stop <container id / container name>
```

- Rm (to remove a container permanently)
```bash
docker rm <container id / container name>
```

- images (to list images)
```bash
docker images

# to delete images
docker rmi <image_name>

!!remember to first delete the containers used by the image you are going to delete!!
```

- pull (to download the image)
```bash
docker pull <image_name>
```


----------------------------------------------------------
**IMPORTANT :**
In case of os images when we run it using "docker run ubuntu" it will not show up in the running containers because the container is not configured to run operating systems so it will exit as soon as it runs. So we can also use **sleep** command to just sleep the container after running
```bash
docker run ubuntu sleep 100
```
----------------------------------------------------------



- Exec (execute a command)
Suppose we need to find the contents of the file in the running image
```bash
# To get the container id / container name
docker ps -a

# We want to see the /etc/hosts file
docker exec <container_name / container_id> cat /etc/hosts
```


### Attach and Detach 
In docker we can run our applications in both attach and detach modes like in detach mode we can run our application in background and vice versa
```bash
# Run a web application for example
docker run <container_name>/<application>

# for running in detach mode
docker run -d <container_name>/<application>

# To attach to the container anytime
docker attach <container_id>
```

### Run Command

- **TAG**

This is to specify the version of the image we need to run or pull in our docker by default it is set to latest
```bash
docker run redis:<version>
```

- **STDIN**

By default docker doesn't allow us to take the input as it is for taking an input we need to specify our stdin (-i = interactive mode) tag to allow inputs and (-t = psuedo terminal mode) to show the prompt
```bash
# This will run in interactive mode
docker run -i <image_name>/<program>

# This will run in terminal mode (will show prompt as well)
docker run -it <image_name>/<program>
```

- **PORT MAPPING**

In docker every container is assigned its own IP and is only accessible from inside or locally in docker host now to make that application accessible to everyone we can use port forwarding and we can also create multiple instances of a single application on different ports.
```bash
# Suppose when we run our application it runs on port 5000
docker run <docker_image_name>/<application>

# PORT FORWARDING
docker run -p 80:5000 <docker_image_name>/<application>
# Now this application will be avialable on port 80 of the docker host's IP
# MULTIPLE INSTANCES
docker run -p 8000:5000 <docker_image_name>/<application>
```

- **VOLUME MAPPING**

In docker if we delete our container our data will get lost so in order to cut that possibility out we will use volume mapping this will map our data in our container with a storage in our docker host. For eg : 
When we run our mysql the data we put in it is stored in container location of /var/lib/mysql now we will map this container with a local folder at path /opt/datadir

```bash
docker run -v /var/lib/mysql:/opt/datadir mysql
```

- **INSPECT**

To get more details about a container
```bash
docker inspect <container_name>
```

- **LOGS**

To display container logs in stdout (if application is running in detached mode)
```bash
docker logs <container_name>
```

### Docker Environment Variables

Now if we need to set some environment variables such as some value or API keys for an application so for that we will set our ENV variable in our docker container Now if we want to run our application using different variables we can use it using -e tag
```bash
# We can also run multiple instances of an application with different ENV variables
docker run -e APP_COLOR=blue <container_name>
docker run -e APP_COLOR=red <container_name>
```


### How to create a Docker image ?

In order to create a docker image we need to create a file in our code folder which will execute the commands in which docker can understand.

```Dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . ./opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

To make our image we will use docker build command and then push to deploy it
```bash
docker build Dockerfile -t <image_name>/<custom_name>
docker push <image_name>/<custom_name>
```

### ENTRYPOINT & CMD

Here while making our Dockerfile we can set our command which we want to run on the system. Now we can set the command using CMD and entrypoint both Now for example we have a Ubuntu machine (ubuntu-sleeper) which is going to sleep for 5 secs and then close so the Dockerfile for this machine will look as :
```Dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

Here we can modify the CMD input which is 5 (hard coded) by using:
```bash
docker run ubuntu-sleeper 10

# here 10 will replace the hard coded 5
```

We can also modify the entrypoint using the (--entrypoint) tag:
```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
Now this will run ubuntu-sleeper with entry point sleep2.0 and CMD variable as 10


### Networking in Docker containers

In docker the containers are given their own IP generally in range of 172.x.x.x and all the containers in a network can access each other and if you want them to be avialable to external hosts we can map the ports using the method discussed above. Some examples :

```bash
# This will run in default network
docker run ubuntu

# This will run the container in none of the network
docker run ubuntu --network=none

# This will run the container directly on the host we don't need to map it to the host ports
docker run ubuntu --network=host
```

#### User Defined Networks

We can also define internal networks according to our need.
```bash
docker network ls
```
OUTPUT:
```bash
NETWORK ID     NAME      DRIVER    SCOPE
0549525f4ad0   bridge    bridge    local
3a74ad9d9a15   host      host      local
c9ff82a29e73   none      null      local
```

Now to create our own network we can use command to create custom network with our own defined subnet.
```bash
docker network create \
  --driver bridge \
  --subnet 182.18.0.0/16
  <custom_network_name>
```

To get network details of a container
```bash
docker inspect <container_name>
# there will be saperate section for it
```

Now if we need to connect two services running in docker so instead of using the docker assigned IP we can simply use container name directly and it is managed by docker itself ->

For Eg : 
```bash
---------------------------
|    HOST    |     IP     |
|--------------------------
|  mysql_A   | 172.x.x.1. |
|--------------------------
|  nodejs1   | 172.x.x.2  |
---------------------------
```

### Storage

In docker when we build a container we cannot edit it further and it becomes a read only layer (image layer). Now if we run the container if forms a read-write layer (container layer) and that layer is modifiable but it's data will be lost if container is crashed or closed. Now the folder structure that docker uses is something like this:

```bash
/var/lib/docker
         |
         ---- /image
         |
         ---- /containers
         |
         ---- /network
         |
         ---- /volumes
         |
         .....
```

This directory is not accessible directly in MacOS so we need to run:
```bash
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

Now if we create a volume in docker it will create a new one in /volumes directory
```bash
docker volume create <volume_name>
```

Now we can map our storage with any container's storage so that our data will get stored in that
```bash
docker run -v <volume_name>:<image_volume_path> <image_name>
```
This whole volume map is known as **Volume Mounting**


Now the another form of mounting is called **Data Mounting / Bind Mounting**
In this situation if I have some data on the docker host and i want to mount it to my docker container then we will use :
```bash
docker run -v /data/<file_name>:<image_volume_path> <image_name>

# using -v is an old way of doing things
# We can use --mount a more prefferable way of doing the same thing

docker run --mount type=bind,source=/data/<file_name>,target=<image_volume_path> <image_name>
```

## Docker Compose

Instead of running the services saperately we can define a docker-compose.yml file which will contain all the configs which we want to run the file as. This file looks as :
```yml
services:
  web:
    image: "<image_name>/<app_folder>"
  database:
    image: "redis"
```
This file above is a version 1 of docker_compose with no network support

https://github.com/dockersamples/example-voting-app

Let's take an exmaple of a simple voting application with 5 services to be linked together :
- Redis DB
- PostgreSQL
- Python Application
- Node Js Application
- .Net service worker

Now if we first write our run commands in order to generate our yml file:
```bash
docker run -d --name=redis redis
docker run -d --name=db postgresql:9.4
docker run -d --name=vote -p 5000:80 --link=redis voting-app
docker run -d --name=result -p 5001:80 --link=db result-app
docker run -d --name=worker --link=redis --link=db
```

Now let's generate the yml file for this 
```yml
redis:
  image: redis
db:
  image: postgresql
vote:
  image: voting-app
  ports:
    - 5000:80
  links:
    - redis
result:
  image: result-app
  ports:
    - 5001:80
  links:
    - db
worker:
  image: worker
  links:
    - redis
    - db
```

Now if we want to first build our image using Dockerfile then we can also use the build option here instead of using the image name. So our new YML file will be something like this:
```yml
redis:
  image: redis
db:
  image: postgresql
vote:
  build: ./vote
  ports:
    - 5000:80
  links:
    - redis
result:
  build: ./result
  ports:
    - 5001:80
  links:
    - db
worker:
  build: ./worker
  links:
    - redis
    - db
```

Now we also have different versions for docker-compose files v1,v2 and v3 :
Major difference between the versions is the support of networks etc.

Our same above file will become as this in version 2 of docker-compose.yml :
```yml
version: 2
services:
  redis:
    image: redis
  db:
    image: postgresql
  vote:
    build: ./vote
    ports:
      - 5000:80
    depends_on:
      - redis
  result:
    build: ./result
    ports:
      - 5001:80
    depends_on:
      - db
  worker:
    build: ./worker
    depends_on:
      - redis
      - db
```

Now if we want to add two networks front-end and back-end in our compose as:
- Redis DB (back-end)
- PostgreSQL (back-end)
- Python Application (front-end)
- Node Js Application (front-end)
- .Net service worker (backend)

```yml
version: 2
services:
  redis:
    image: redis
    networks:
      - back-end
  db:
    image: postgresql
    networks:
      - back-end
  vote:
    build: ./vote
    ports:
      - 5000:80
    depends_on:
      - redis
    networks:
      - back-end
      - front-end
  result:
    build: ./result
    ports:
      - 5001:80
    depends_on:
      - db
    networks:
      - back-end
      - front-end
  worker:
    build: ./worker
    depends_on:
      - redis
      - db
    networks:
      - back-end

networks:
  - back-end
  - front-end
```


## Docker Registery

Now we must be thinking about where do we get our docker images when we pull or run our image. 
For eg:
```bash
docker pull nginx
```
Here let's elaborate **nginx**

originally the image looks like :

```bash
docker.io/nginx/nginx
# Registry/(User,Account)/(Image,Repository)
```

docker.io is the default Registery for all the images we pull suppose we want to get image other than the docker.io registery we have to write the full address unlike when pulling from the docker hub.
For eg:

```bash
docker pull gcr.io/kubernetes-e2e-test-images/dnsutils
```

### Private Registery

If we want to host our private images and we don't ant anyone to access it docker provides us with private registeries that we can use which are provided with Azure and GCD platform

Now to Access our private registery 
```bash
docker login private-registery.io
docker run private-registory.io/apps/internal-app
```

### Deploy Your Own Private Registery

Now if we want to deploy our own registery in our own docker host we can do it by using the following steps:
```bash
# Running our registery with a port mapping to the host
docker run -d -p 5000:5000 --name registery registery:2

# Now we will tag our image with the registery running
docker image tag my-image localhost:5000/my-image

# Pushing our image
docker push localhost:5000/my-image
```

If we want to pull the image we can do it by
```bash
# If we want to pull image locally
docker pull localhost:5000 my-image

# If we want to pull image from within the network but from different host
docker pull 192.168.56.100:5000/my-image
```

## Docker Engine

Docker engine is the engine running in the system that consist of 
- Docker CLI
- Docker REST API
- Docker Deamon

Here CLI is what we use and Docker deamon is the process that runs with the shared resources of the system and REST API connects the both services.

Now we can access the docker CLI from anywhere just by specifying the -H tag
```bash
docker -H=10.134.23.90:3450 run nginx
```

Now in docker the application runs own it's own by specifying the root PID to 1 but when our system runs the PID 1 is already taken so what happens is our container maps it's processes with the processes outside the container to avoid any errors and giving the independency of our host. We can also limit our resources to the container by using specific commands while running the image

```bash
# Not to exceed the CPU above 50%
docker run --cpus=.5 ubuntu

# Not to exceed the Memory 100 MB
docker run --memory=100m ubuntu
```

## Container Orchestration

If we need a stable application we need to ensure that our application is running all the time we need to make sure that our host is always reachable and If one image crashes then it should be replaced by the another instance instantly. So solution for this problem is :
- Docker swarm
- Mesos
- Kubernetes



## Docker Swarm

Docker swarm can produce hundreds or thousands of replicas of your container on multiple hosts :
```bash

# Write this in master host
docker swarm init

# This command will provide us a command which we have to write in our worker nodes
```

Now to run the services in a swarm
```bash
docker services create --replicas=3 my-web-server
```
