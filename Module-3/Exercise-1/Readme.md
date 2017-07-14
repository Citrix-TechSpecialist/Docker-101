# Create a docker-compose.yaml File

Instead of creating a `docker-compose.yml` file from scratch, we are going to copy one from another repository to get started. We will then examine the file and understand it's anatomy before finally making edits to suite our environment needs. 

### Step 1

To get started, enter the following commands to clone a repository with a `docker-compose.yml` file already made for us. Navigate to the directory and view the contents with `nano`. 

```
# Change directory to the workspace you want to clone the repository
cd /data

# Clone the desired repository 
sudo git clone https://github.com/Citrix-TechSpecialist/nitro-ide.git

# enter the directory of the repository
cd nitro-ide

# View the file contents of docker-compose.yml
nano docker-compose.yml
```
Here is a copy of the [docker-compose.yml](./scripts/docker-compose.yml) file for reference. It is recommended you open it in another tab in your browser to follow along.  

Below are the desired services we want to configure and deploy.

  1. [Webserver A](https://hub.docker.com/r/mayankt/webserver/) that is a static website site
  2. [Webserver B](https://hub.docker.com/r/mayankt/webserver/) that is a different static website
  3. [Cloud9 IDE](https://c9.io/) which we will use to write code and execute python scripts to automate configuration of NetScaler CPX. 
  4. [NetScaler CPX](https://microloadbalancer.com) a NetScaler in a docker container that share the same API as other NetScaler ADCs.  

### Explaining the docker-compose.yml File

Below are snippets of the `docker-compose.yml` with comments (`#`) per line with details of the `key` : `value` pairs describing the desired deployment. 

### Sandbox Network

With Docker you can define specific container networks. In this case we are creating a [bridge network](https://docs.docker.com/engine/userguide/networking/#bridge-networks) specific to deploying only our desired containers to within a SDN boundary internal to the host. 

```
networks:                             # This defines that below are settings for docker networks
  sandbox:                            # Name of the network
    driver: bridge                    # The type of network driver to use 
    ipam:                             # Details of the network and IP space
      config:                         # Configuration parameters 
        - subnet: "192.168.13.0/24" 	# The desired subnet of the docker network
```


### WebServer A / B 

```
webserver-a/b:  						# Service name
    image: "mayankt/webserver:a" 		# Docker container image to use
    restart: always 					# Restart the service if it fails or the host reboots
    networks: 							# This describes the docker networks the containers will be part of
      sandbox: 							# Docker network's name
        ipv4_address: "192.168.13.11" 	# Static IP address of this service/container. You can leave this key:value out and to obtain an IP from Docker's IPAM
    hostname: webserver-a 				# Desired hostname of the container
```

### NetScaler CPX 

```
  cpx:												# Service Name
    image: "store/citrix/netscalercpx:12.0-41.16"	# Docker container image to use from Citrix' registery
    environment:									# Environment Variables local to the container
      EULA: "yes"									# same as export EULA="yes" as a pre-req for CPX to work
    restart: always									# Restart the service if it fails or the host reboots
    cap_add:										# Add specific container kernel capabilities https://docs.docker.com/engine/security/security/#linux-kernel-capabilities
      - NET_ADMIN									# Perform various network-related operations https://linux.die.net/man/7/capabilities
    ulimits:										# Override the default (resource) ulimits for a container	
      core: -1										# Use unlimited CPU, up to the amount available on the host system.
    networks:										# This describes the docker networks the containers will be part of
      sandbox:										# Docker network's name
        ipv4_address: "192.168.13.20"				# Static IP address of this service/container. You can leave this key:value out and to obtain an IP from Docker's IPAM
    ports:											# Exposed ports mapped to the host from the container. 
      - "10000-10050:10000-10050"					
      - "9080:80"
    hostname: ns-adc								# Desired hostname of the container
```

### Cloud9 IDE

```
  nitro-ide:										# Service Name
    image: "mayankt/nitro-ide"						# Docker container image to use from Citrix' registery
    restart: always									# Restart the service if it fails or the host reboots
    dns: 8.8.8.8									# Specific DNS server to use for host name resolution from within the container
    networks:										# This describes the docker networks the containers will be part of
      sandbox:										# Docker network's name
        ipv4_address: "192.168.13.10"				# Static IP address of this service/
    ports:											# Exposed ports mapped to the host from the container. 
      - "9090:80"
      - "9091:8000"
    links:											# Link to containers in another service given service name and/or a link alias ("SERVICE:ALIAS"). "ping web-a" will ping the webserver-a service from within the container.
      - "cpx"
      - "webserver-a:web-a"
      - "webserver-b:web-b"
    volumes:										# Volume mounts local to the host mapped to a directory local to the container with read/write access (rw)
        - ${DATA_DIR}:/workspace:rw      
    hostname: nitro-ide								# Desired hostname of the container
```
> Note you may have to uncomment the `volumes` section to mount volumes in the docker file. 

### Step 2

Set the environmental variable `DATA_DIR` to `/data` on the docker host. This environment variable will substitute the value `/data` into the docker compose file when we provision our containers. Type the following on your docker host: 

```
export DATA_DIR="/data"`
```

Verify that the environmental variable was set successfully by typing the following command: 

```
echo $DATA_DIR
```

It should return the `/data` directory path. 

### Conclusion

In this module we clones a repository with our desired compose file. We explored what constitutes a `docker-compose.yml` file and what the various parameters mean. We set the value `/data` for a placeholder in the compose file that took in an environment variable to specify which local directory will be mapped to our IDE's local workspace `/workspace` so we can share data from host to container. 

Here is an overview of configuration steps: 

![docker-compose.yml](./images/docker-compose.gif)

Now continue on to [Exercise-2](../Exercise-2) to compose your docker environment. 

### Shortcuts

1. [Module 0-A: Install Docker Locally](https://hub.docker.com/?next=https%3A%2F%2Fhub.docker.com%2F)
2. [Module 0-B: Access your Docker Lab Development Box](../../Module-0)
2. [Module 1: Running Docker Containers](../../Module-1)
3. [Module 2: Creating Custom Images from Dockerfiles](../../Module-2)
4. [Module 3: Using Docker Compose](../../Module-3)

