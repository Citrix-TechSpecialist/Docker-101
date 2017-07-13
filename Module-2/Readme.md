# Module 2: Create an Image from a Dockerfile

Given that all Docker containers are based off specific [Docker images](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/), in this module we will explore one of many ways to create Docker Images that define your custom docker containers. The following list a few ways to create customer images docker images: 

1. [Save a running container into an image](https://docs.docker.com/engine/reference/commandline/save/)
  
    * In this method, users initially run a docker container based on a desired base image. Then after changes are made by `docker exec -it` or other means on the running container, the live container is ultimatly saved into a `.tar` archive in its desired state that can be later [loaded](https://docs.docker.com/engine/reference/commandline/load/) to run multiple new instances of the custom image. 

2. Defining a [Dockerfile](https://www.digitalocean.com/community/tutorials/docker-explained-using-dockerfiles-to-automate-building-of-images)
  
    * This method is the most robust and more commonly use method to define and create docker images. A Dockerfile is essentially a recipe of commands to execute on a defined base image that constitute the desired state of a custom docker image. Consider the traditional workflow to configure a webserver which requires executing of scripts, updating local packages, pulling of code from various repositories, installing dependencies, etc. before the webserver is in its desired state. A Dockerfile can define those steps towards a desired state thus allowing changes to be made independently on external packages, code repositories, etc. and the Dockerfile will simply execute those commands and operations when you desire to build your custom container image. 

## Dockerfiles

The advantage of a Dockerfile over just storing the binary image (or a snapshot/template in other virtualization systems) is that the automatic builds will ensure you have the latest version of code, packages, and external resources available in your docker container. This is a good thing from a security perspective, as you want to ensure you’re not installing any vulnerable software. This is also a good thing from an operational perspective because it allows you to rapidly build out isolated environments based on defined recipe that can pull from external resources like git to compile and build microservices. 

Below is an example of a simple [Dockerfile](https://github.com/Citrix-TechSpecialist/GoLang-cpx): 

```
FROM fnichol/uhttpd
MAINTAINER Mayank Tahilramani and Brian Tannous
COPY ./cpx-blog /www
EXPOSE 80
ENTRYPOINT /tmp/update.sh && /usr/sbin/run_uhttpd -f -p 80 -h /www
CMD [""]
```
Breakdown for details below: 

```
FROM fnichol/uhttpd
```

 This denotes the base image to use. In this case, it's an image from Docker Hub of user [`fnichol`](https://github.com/fnichol/docker-uhttpd) who has already made a bare bone minimalistic docker image with the service [httpd](https://httpd.apache.org/docs/2.4/programs/httpd.html) pre-installed which allows us to hosts websites. All we have to do is provide our HTML code and relevant data. From this image, we will make changes and define our custom image based on subsequent commands in our Dockerfile. 
    
  * **FROM** is a mandatory declaration in a Dockerfile.

```
MAINTAINER Mayank Tahilramani and Brian Tannous
```

 This is just meta data for the image on who the maintainer/creator of the image and Dockerfile are. 
    
  * **MAINTAINER** is an optional declaration in a Dockerfile.

```
COPY ./cpx-blog /www
```

  This command simply copies everything `cpx-blog` directory that is local to the Dockerfile into the `/www` directory that is local to the contianer. Within the container there must already be a `/www` directory (as specified by the [base image](https://github.com/fnichol/docker-uhttpd) to put content in. In this case, any html code or data that will be served by httpd must reside in the `/www` directory within the container. 

```
EXPOSE 80
```

  This command simply states that port 80 will be open on the container as expected to host a website.

```
ENTRYPOINT /usr/sbin/run_uhttpd -f -p 80 -h /www
```
 This command dictates what to execute when the container starts. Note that the container's lifespan is directly dependent on the service it runs on start, in our case the httpd (found at `/usr/sbin/run_uhttpd`) is executed. The entrypoint script basically starts the webservice `httpd` hosting content in `/www`. If for whatever reason the uhttpd service itself fails, hangs, or stops, the running container will stop running as well.

`CMD [""]`
 This command is similar to ENTRYPOINT where traditionally you would define the default command to execute when the container starts. In this case, our ENTRYPOINT script is handling that for us so CMD can be left empty. 
   * CMD is a mandatory declaration in a Dockerfile. 

## Exercises 

Navigate to and complete the following exercises within Module 2:

1. [Build a Docker Image](./Exercise-1)
2. [Running the Container](./Exercise-2)

# Reset the Sandbox Environment 

Once completed Module 1, reset the environment by typing in the following command in your sandbox environment: 

`sudo /dockerclean.sh`

If you are following along on your local machine, enter the following commands to remove all docker containers, images, and docker volumes from your host: 

```
docker kill $(docker ps -q)
docker rm -f $(docker ps -a -q)
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -qf)
```

### Shortcuts

[Table of Contents](../)
[Module 1](../Module-1)
[Module 3](../Module-3)
