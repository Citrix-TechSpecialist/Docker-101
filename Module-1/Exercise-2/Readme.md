# Running a Container

Lets convert a docker image into a running instance of a docker container to host a simple website. 

### Step 1

We will issue [`docker run`](https://docs.docker.com/engine/reference/run/) commands to run containers. Type the following command to host a website on the docker host on port `10000`.  

`docker run -dt --restart=always --name=cpx-blog -p 10000:80 mayankt/cpx-blog`

Here is the breakdown of the command from above: 

* `docker run -dt` 
    * This will run the container detached in the background. Later we will see [how we can attach to this container's terminal](https://stackoverflow.com/questions/30172605/how-to-get-into-a-docker-container), but for now we will have the container running detached in the background as a daemon. 

* `--restart=always`
    * This will restart the container automatically if it crashes or if and when docker/host restart.

* `--name=cpx-blog`
    * This gives the container a name for more intuitive reference in later docker commands. Without a name parameter, the container will be randomly assigned a name and can be referenced to via the random name or the hash id of the container. 

* `-p 10000:80` 
    * This will expose port `10000` on the host and map it to port `80` on the container for access to the hosted website.

* `mayankt/cpx-blog`
    * This identifies the `latest` tagged image by default because no explicit tag is specified. It will be pulled from dockerhub to use when running the container.

    > It is not necessary to pull a desired image before executing a `docker run` command. If the image does not exist locally, it will be automatically pulled from the designated registry (Docker Hub in our case). 

Once you have entered the command, you will notice an output similar to below: 

```
Unable to find image 'mayankt/cpx-blog:latest' locally
latest: Pulling from mayankt/cpx-blog
3ac0c2aa6889: Already exists 
ec2ec713dc4f: Already exists 
ea0a5af9851c: Already exists 
555bf6439b47: Already exists 
71080d75d6eb: Already exists 
c787ac6d0b0a: Already exists 
1a9841bc3a47: Already exists 
336c032aec9b: Pull complete 
bc3b4209c6c5: Pull complete 
3a5d33d6e1e0: Pull complete 
7e846adb4c7d: Pull complete 
Digest: sha256:141100857249a391261edf7335ffea1ca20478a15d3ac08c821561e7a8998ef9
Status: Downloaded newer image for mayankt/cpx-blog:latest
222bcc5747008a5c05f79d3a717f9132c8aa66234939677d3ebbcc5d883b5b5c
```
   > Notice some of the layers already exist locally from our previous webserver-a pull because both images use mutual base images that can be recycled and save on local space. Only deltas or layers that make cpx-blog unique from webserver-a are pulled that are not stored locally. Also the output provided in the last line `222bcc5747008a5c05f79d3a717f9132c8aa66234939677d3ebbcc5d883b5b5c` is the long unique id of the running container that can be referenced in future docker commands alternative to `cpx-blog`. 

### Step 2

To see all of your running containers type the following command:

`docker ps`

That will yield an output similar to below.

```
CONTAINER ID        IMAGE               COMMAND                CREATED              STATUS              PORTS                            NAMES
222bcc574700        mayankt/cpx-blog    "/bin/sh -c 'nginx'"   About a minute ago   Up About a minute   443/tcp, 0.0.0.0:10000->80/tcp   cpx-blog
```

This shows that the container named cpx-blog is running based on the image mayankt/cpx-blog with an external 10000 port exposed on the local host for remote access. Navigate to [`htt://localhost:10000`](http://localhost:1000) to see the website. 

  > If you are following along in the sandbox environment, navigate to [`http://userX-lb.sl.americasreadiness.com`](http://userX-lb.sl.americasreadiness.com) which is a proxied VIP provided for you for remote access. Remember, X denotes your user number. 

To stop your running container, type the following command: 

`docker stop cpx-blog`

Now to see all of your running and **non running** containers, type the following command: 

`docker ps -a`

That will yield an output similar to below. 

```
CONTAINER ID        IMAGE                 COMMAND                CREATED             STATUS                       PORTS               NAMES
f407d06318f0        mayankt/cpx-blog   "/bin/sh -c 'nginx'"   10 minutes ago      Exited (137) 3 seconds ago                       cpx-blog
```

### Step 3

If you container is not running already, start the container with the following command: 

`docker start cpx-blog`

Then type the following command to enter the terminal shell of the container: 

`docker exec -it cpx-blog /bin/sh`

The breakdown of the command as follows: 

* `docker exec -it` 
    * This will execute a command on the container with an interactive terminal (`-it`). We could execute commands in the background with a `-dt` if we wanted to, but we wouldn't see the outcome and the command we're running in this example is invoking a shell. If we wanted to run a script and not interact with it, we could have used the `-dt` flag.  

* `cpx-blog-b`
    * This denotes to which running container to execute the command against. In our case, it's the cpx-blog container.

* `/bin/sh`
    * This is the command itself that we want to execute in the container. In this example we are invoking the shell for terminal access into our container. In other examples we could execute specific scripts or bash files as well (i.e. `/var/scripts/init-db.sh` or any other file locally stored within the container).

After executing the command, you should have entered into the shell prompt of the container itself. Lets manipulate some files inside the container and see them reflected onto the website. 

Within the container shell, enter the following: 

`vi /var/wwww/index.html`
  > `vi` invokes a CLI text editor. It is functionalyl the same thing as notepad or sublime text, but geared for CLI interfaces not GUI interfaces. `vi` utility helps edit files in the terminal.

Once you have opened the `index.html` file, scroll down and change the text between : 

` <h1 class="brand-title">NetScaler NITRO Blogs</h1>  ` to anything you desire, such as ` <h1 class="brand-title">LEARN DOCKER!</h1>  `

  > Edit text by entering the `i` key and then navigating the cursor to delete text you want to remove then typing in text you want to replace it with. Once completed, hit the `esc` key then `shift` + `:` key. Type `wq` to write and quit. You can then hit `ctrl` + `l` to clear the screen if you desire. 

Once back into the container's CLI, simply type `exit` to logout and return back to the host's terminal. 

Once the `index.html` is updated, refresh your browser or navigate back to [`http://localhost:1000`](http://localhost:1000) to observer changes to your site.
  > If you are following along in the sandbox environment, navigate to [`http://userX-lb.sl.americasreadiness.com`](http://userX-lb.sl.americasreadiness.com) which is a proxied VIP provided for you for remote access. Remember, X denotes your user number. 

### Conclusion

This concludes how to run a container with `docker run` command, how to attach to a container's terminal via the `docker exec -it` command, and how to manipulate data within a running container. 

![docker-run](images/docker-run.gif)

### Shortcuts

1. [Module 0-A: Install Docker Locally](https://hub.docker.com/?next=https%3A%2F%2Fhub.docker.com%2F)
2. [Module 0-B: Access your Docker Lab Development Box](../../Module-0)
2. [Module 1: Running Docker Containers](../../Module-1)
3. [Module 2: Creating Custom Images from Dockerfiles](../../Module-2)
4. [Module 3: Using Docker Compose](../../Module-3)

