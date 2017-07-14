# Build a Docker Image

In this exercise we will create a new docker image locally from a Dockerfile as discussed in [Module 2](../) overview. We will also make edits to the Dockerfile to cusomize the image to our preference. 

### Step 1

If you're following along in the sandbox environment, [logon to the console](../../Module-0) otherwise follow along on your local machine with docker installed. 

In the `/data` directory on your host, clone the following git repository: 

```
# Change the working directory to /data on the docker host
cd /data

# Clone a copy of the github project locally
sudo git clone https://github.com/Citrix-TechSpecialist/GoLang-cpx.git

# List the contents in the /data directory to see the GoLang-cpx project directory
ls -l
```

in the `/GoLang-cpx` directory there is a `Dockerfile`. Lets view the contents with the `cat` command. 

```
# Change working directory into the GoLang-cpx project
cd GoLang-cpx

# View what all is in the GoLang-cpx directory
ls -l

# View the Dockerfile contents
cat Dockerfile
```

Once you `cat` the file, you will notice the following content in the [Dockerfile](./scripts/Dockerfile): 

```
FROM fnichol/uhttpd
MAINTAINER Mayank Tahilramani and Brian Tannous
COPY ./cpx-blog /www
EXPOSE 80
ENTRYPOINT /usr/sbin/run_uhttpd -f -p 80 -h /www
CMD [""]
```
The content above is pretty much the same we observed in the [Module-2](../) overview. We will now build this container using the [`docker build`](https://docs.docker.com/engine/reference/commandline/build/) command. 

# Step 2

Type the following command to build your docker image: 

```
docker build -t cpx-blog .
```

Here is the breakdown of the command above: 

`docker build` basically tells the docker engine to build an image.

`-t` give the image a name and optionally a tag in the `name:tag` format.

`cpx-blog` will be the name of the created docker image.

`.` tells docker engine to look for a file named *Dockerfile* (by default) in the current directory. 

You will see the following output: 

```
Sending build context to Docker daemon  1.773MB
Step 1/6 : FROM fnichol/uhttpd
latest: Pulling from fnichol/uhttpd
a3ed95caeb02: Pull complete 
1775fca35fb6: Pull complete 
718e21306e6b: Pull complete 
889bfeab2d4e: Pull complete 
8ac43f1732b7: Pull complete 
cefd08b5f834: Pull complete 
a32be2ed7953: Pull complete 
1c78be7a5ec7: Pull complete 
74984e6e6d1c: Pull complete 
Digest: sha256:28e6f95cf33ae1336525034e2b9d58ddf3cc63a2cdd9edebc8765321d96da9e0
Status: Downloaded newer image for fnichol/uhttpd:latest
 ---> df0db1779d4d
Step 2/6 : MAINTAINER Mayank Tahilramani and Brian Tannous
 ---> Running in 459db6d6a053
 ---> d51effaba5ae
Removing intermediate container 459db6d6a053
Step 3/6 : COPY ./cpx-blog /www
 ---> b1510a20020d
Removing intermediate container b075c62a629b
Step 4/6 : EXPOSE 80
 ---> Running in 84f5263cb817
 ---> f1f4672c9d5b
Removing intermediate container 84f5263cb817
Step 5/6 : ENTRYPOINT /usr/sbin/run_uhttpd -f -p 80 -h /www
 ---> Running in 8e2b276c3aa9
 ---> 552dfe1be7a7
Removing intermediate container 8e2b276c3aa9
Step 6/6 : CMD 
 ---> Running in 7c5da4990bea
 ---> 0c78968bbe81
Removing intermediate container 7c5da4990bea
Successfully built 0c78968bbe81
Successfully tagged cpx-blog:latest
```

You can also see the images on the local host with the following command: 

`docker images`

which shows:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
cpx-blog            latest              0c78968bbe81        46 seconds ago      5.66MB
fnichol/uhttpd      latest              df0db1779d4d        3 years ago         4.87MB
```
Docker pulled the base image `fnichol/uhttpd` from Docker Hub as well as created the layer on top with the changes we added defined in our Dockerfile to create the `cpx-blog` image. This concludes the creation of the image. Next we will create *ONE* more image that is an updated version of the `cpx-blog` image before we run our containers in [Exercise-2](../Exercise-2). 

# Step 2

Lets make updates to our `Dockerfile` in `/data/GoLang-cpx`. Open the file up in `nano` with the following command. `nano` is nothing more than a simple text editor for CLI. It can be considered an equivalent to [Sublime Text](https://www.sublimetext.com/) or [Notepad++](https://notepad-plus-plus.org/).

```
# In the /data/GoLang-cpx directory enter the following command:
sudo nano Dockerfile
```

Update the file with your cursor and keyboard to reflect the following: 

```
FROM fnichol/uhttpd
MAINTAINER Mayank Tahilramani and Brian Tannous
COPY ./cpx-blog /www
WORKDIR /www
RUN echo "find /www -type f -exec sed -i \"s/All rights reserved./Hosted by container: ${HOSTNAME}/g\" {} \\;" > /tmp/update.sh && chmod +x /tmp/update.sh
EXPOSE 80
ENTRYPOINT /tmp/update.sh && /usr/sbin/run_uhttpd -f -p 80 -h /www
CMD [""]
```
    >To exit `nano` after you are done editing, enter the keys `ctrl` + `x` then `y` and `enter` to save and quit. 

Here are the details on the updated command lines added above: 

```
WORKDIR /www
```

  This changes the working directory of the docker build engine when executing `RUN` commands in the next line. Note that you cannot simply change directories by a single `RUN` command for example `cd /www` because each time you execute a `RUN` command, docker spawns a new container and therefore the default working directory become `/`. See more context [here](https://stackoverflow.com/questions/17891981/docker-run-cd-does-not-work-as-expected) 


```
RUN echo "find /www -type f -exec sed -i \"s/All rights reserved./Hosted by container: ${HOSTNAME}/g\" {} \\;" > /tmp/update.sh && chmod +x /tmp/update.sh
```
  This command simply creates a script `/tmp/update.sh` which finds all files in the `/www` directory and replaces strings in them which match the pattern "*All rights reserved.*" with the string "*Hosted by container: ${HOSTNAME}*" where `${HOSTNAME}` is a built in environmental variable in the running container that holds the container's unique host name. By default the hostname of any container is it's short uuid. This script will essentially replace the footer of all web pages which state "All rights reserved" with which container is specifically hosting the website.
   
     >Note that this command creates a script `/tmp/update.sh` but does not execute it. This command needs to be executed in a final running container state, not during an intermediate step when building the final image. Hence this script created here is executed in the `ENTRYPOINT` step below.

```
ENTRYPOINT /tmp/update.sh && /usr/sbin/run_uhttpd -f -p 80 -h /www
```
 This command dictates what to execute when the container starts. Note that the [container's lifespan](https://medium.com/@lherrera/life-and-death-of-a-container-146dfc62f808) is directly dependent on the service it runs on start, in our case first the `/tmp/update.sh` script executes to update footers on all html pages, then the httpd (found at `/usr/sbin/run_uhttpd`) is executed. The entrypoint script basically starts the webservice hosting content in `/www`. If for whatever reason the uhttpd service itself fails, hangs, or stops, the running container will stop as well.

With the updated changes in our Dockerfile, we have introduced a new script into the container `/tmp/update.sh` which updated some HTML text on our website and is executed upon running the container as defined by our `ENTRYPOINT` statement. 

Now lets build our new image: 

```
# In the /data/GoLang-cpx directory enter the following command:
docker build -t cpx-blog:v2 .
```
View your images via the `docker images` command to see a new tagged version of the `cpx-blog` image was created. 

```
WORKDIR /www
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
cpx-blog            v2                  7f4074f9eecf        9 seconds ago       5.66MB
cpx-blog            latest              0c78968bbe81        25 minutes ago      5.66MB
fnichol/uhttpd      latest              df0db1779d4d        3 years ago         4.87MB
```

### Conclusion 

In **Step 1** and **Step 2** we created 2 docker images of the same CPX-blog website. The first image was created using a Dockerfile form the [GoLang-cpx](https://github.com/Citrix-TechSpecialist/GoLang-cpx/) repository. The second image was created from a custom Dockerfile which added a script to update the footer of the website with the container's hostname. Below is a overview of the steps above. 

![docker build](./images/docker-build.gif)

Now lets move on to [Module-2: Exercise 2](../Exercise-2) to run the containers from the images we created. 

### Shortcuts

1. [Module 0-A: Install Docker Locally](https://hub.docker.com/?next=https%3A%2F%2Fhub.docker.com%2F)
2. [Module 0-B: Access your Docker Lab Development Box](../../Module-0)
2. [Module 1: Running Docker Containers](../../Module-1)
3. [Module 2: Creating Custom Images from Dockerfiles](../../Module-2)
4. [Module 3: Using Docker Compose](../../Module-3)
