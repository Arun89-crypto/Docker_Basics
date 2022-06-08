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

In docker when we build a container we cannot edit it further and it becomes a read only layer (image layer). Now if we run the container if forms a read-write layer (container layer) and that layer is modifiable but it's data will be lost if container is crashed or closed. Now the folder structure that docker uses is something like this 

```bash
/var/lib/docker
         |
         ---- /image
         |
         ---- /containers
         |
         ---- /network
         |
         .....
```


