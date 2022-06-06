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
