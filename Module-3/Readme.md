# Module 3: Using Docker Compose

By this point, it is assumed you have completed [Module 1](../Module-1) and [Module 2](../Module-2) and are familiar with basic Docker commands such as [`docker run`](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems), [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/), [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/), [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm/), and the various parameter flags (such as `-v` for volume mounts) associated with some of these commands. 

It should also be obvious at this point that deploying docker containers at scale by hand with `docker run` commands can be very involved and, at time, too complicated with multiple lines of `docker ..` commands to deploy a large environment. Luckily, docker containers are not meant to be deployed via individual commands, rather they are often deployed to a desired state using various other tools that help automate and/or orchestrate microservices backed by docker containers. Some of these accompanying tools are provided below for reference. 

* Kubernetes -- Google's container orchestration and automation solution to schedule and maintain service state of docker containers. 
* [Mesos](https://mesosphere.com/why-mesos/?utm_source=adwords&utm_medium=g&utm_campaign=43843512431&utm_term=mesos&utm_content=196225818929&gclid=CjwKEAjwtJzLBRC7z43vr63nr3wSJABjJDgJ_9xn3RWHnkH_nDjxQs1X8U6YgQ0drZPoOTfLv9-4hhoCqN3w_wcB)/[Marathon](https://mesosphere.github.io/marathon/) -- Another Automation platform (Mesos) with an orchestration framework (Marathon) to ensure service state of docker containers. 
* [Rancher](http://rancher.com/) -- Opensource container management solution that makes it easy to deploy and manage containers in their own 'Cattle environments' and can even operate and manage other orchestration platforms like Kubernetes, Mesos/Marathon, and Docker Swarm. 
* [Docker Swarm](https://docs.docker.com/swarm/overview/) -- Docker's solution to automation and orchestration of clustered resources to provide a pool of Docker hosts into a single, virtual Docker host. 
* [Docker Compose](https://docs.docker.com/compose/) -- A automation tool for defining and running multi-container Docker applications. This tool is less sophisticated than the ones listed above and more simpler to use but with fewer features for larger deployments at scale.

**In this module, we will be focusing on learning how to use Docker Compose to provision a self contained development environment based on a single input file that describes our desired state and configuration.**

## Docker Compose 

>Source of description comes from [Docker's documentation](https://docs.docker.com/compose/overview/).

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application’s services. Then, using a single command, you create and start all the services from your configuration.

Using Compose is basically a three-step process.

**Step 1**: Define your container with a `Dockerfile` so it can be reproduced anywhere. Either have provide the [Dockerfile as an input](https://docs.docker.com/compose/reference/build/) or have the defined container hosted in a [docker registry](https://docs.docker.com/registry/) like [docker hub](hub.docker.com/). 

**Step 2**: Define the containers that make up your microservices in a `docker-compose.yml` file so it can be run together with other containers in an isolated environment. The `docker-compose.yml` basically consist of `key` : `value` pairs as per the [yaml syntax](https://github.com/Animosity/CraftIRC/wiki/Complete-idiot's-introduction-to-yaml) describeing the desired state of your services. 

**Step 3:** Lastly, run the command `docker-compose up` and Compose will start and run your entire microservice based app as per the desired state.

# Overview 

In this module, we are going to automate the deployment of a simple, self contained, dockerized [sandbox environment](https://github.com/Citrix-TechSpecialist/nitro-ide) to write scripts that issue [NITRO](http://docs.citrix.com/ja-jp/netscaler/11/nitro-api.html) commands to your NetScaler ADCs. In this case we will be issuing commands to a [NetScaler CPX](microloadbalancer.com) that will be locally provisioned on your machine to load balance simple containerized websites. However, it should be noted that this tutorial can be translated to develop and issue commands against other [NetScaler ADCs](https://www.citrix.com/products/netscaler-adc/platforms.html) as well if desired. 

The desired environment will have the following topology: 

![Nitro Dev-box topology](./images/topology.jpg) 

**Services include: **

  * **Webserver A** which is a static containerized HTTP website

  * **Webserver B** which is another static containerized HTTP website

  * **NetScaler CPX** which will be the target NetScaler to send NITRO API calls to load balance webserver A and webserver B.
  
  * **Cloud9 IDE** which is web-based Interactive Developer Environment that allows for rapid scripting and coding through a web browser.

  >All the services above will be isolated in a dedicated [Docker Network](https://docs.docker.com/engine/userguide/networking/). Individual web interfaces that we will need direct external access to will have [external ports mapped](https://docs.docker.com/compose/compose-file/compose-file-v2/#ports) to the container for access from the underlay network (basically your host's LAN).

## Pre-requisite: Reset your environment before continuing. 

Reset the environment by typing in the following command in your sandbox environment: 

`sudo /dockerclean.sh`

If you are following along on your local machine, enter the following commands to remove all docker containers, images, and docker volumes from your host: 

```
docker kill $(docker ps -q)
docker rm -f $(docker ps -a -q)
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -qf)
```

## Exercises 

Navigate to and complete the following exercises within Module 3:

1. [Create a docker-compose.yaml File](./Exercise-1)
2. [Compose an Environment](./Exercise-2)

### Shortcuts

[Table of Contents](../)
[Module 1](../Module-1)
[Module 2](../Module-2)
