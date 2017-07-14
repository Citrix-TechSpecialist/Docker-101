# Compose an Environment

Once you have your [docker-compose.yml](../Exercise-1/scripts/docker-compose.yml) set, you can move forward with provisioning your environment. 

### Step 1

In the `/data/nitro-ide` directory, enter the following commands: 

```
# Navigate to the repository local to your host
cd /data/nitro-ide

# Issue the docker compose command to provision your environment
docker-compose up -d 
```
  > The `-d` in the [`docker-compose up -d`](https://docs.docker.com/compose/reference/up/) specifies that containers run in the background in detached mode. 

You should observe an output similar to the following: 

```
Pulling cpx (store/citrix/netscalercpx:12.0-41.16)...
12.0-41.16: Pulling from store/citrix/netscalercpx
4e1f679e8ab4: Pull complete
..
..
..
588f5003e10f: Pull complete
Digest: sha256:31a65cfa38833c747721c6fbc142faec6051e5f7b567d8b212d912b69b4f1ebe
Status: Downloaded newer image for store/citrix/netscalercpx:12.0-41.16
Pulling nitro-ide (mayankt/nitro-ide:latest)...
latest: Pulling from mayankt/nitro-ide
a3ed95caeb02: Pull complete
..
..
..
9581fa7fd579: Pull complete
Digest: sha256:53c464876633e95f8e11ea821c50add0ff8e00a70c5aacd65f465d2d3045d8d3
Status: Downloaded newer image for mayankt/nitro-ide:latest
Pulling webserver-b (mayankt/webserver:b)...
b: Pulling from mayankt/webserver
3ac0c2aa6889: Pull complete
..
..
..
4484f1613730: Pull complete
Digest: sha256:5807d78ba9c3892238a1eef2763c82f719d077b02a0c087122b816d276f0fbc4
Status: Downloaded newer image for mayankt/webserver:b
Pulling webserver-a (mayankt/webserver:a)...
a: Pulling from mayankt/webserver
3ac0c2aa6889: Already exists
..
..
..
f128b2a739b4: Already exists
1341f98ff817: Pull complete
Digest: sha256:921d4054855c335dcd48a83bd881fa9059fa003f62f1b29bbe4b3a40fc79cc9a
Status: Downloaded newer image for mayankt/webserver:a
Creating nitroide_webserver-b_1
Creating nitroide_nitro-ide_1
Creating nitroide_cpx_1
Creating nitroide_webserver-a_1
```
You can validate your desired containers are running by issueing a `docker ps` command to see all running containers.  

```
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS             
                                                                     NAMES
37892600b3d6        store/citrix/netscalercpx:12.0-41.16   "/bin/sh -c 'bash ..."   13 seconds ago      Up 10 seconds       22/tcp, 443/tcp, 1
61/udp, 0.0.0.0:10000-10050->10000-10050/tcp, 0.0.0.0:9080->80/tcp   nitroide_cpx_1
772b633440d7        mayankt/webserver:a                    "/bin/sh -c 'nginx'"     13 seconds ago      Up 10 seconds       80/tcp, 443/tcp   
                                                                     nitroide_webserver-a_1
aeef73f08b84        mayankt/nitro-ide                      "supervisord -c /e..."   13 seconds ago      Up 11 seconds       3000/tcp, 0.0.0.0:
9090->80/tcp, 0.0.0.0:9091->8000/tcp                                 nitroide_nitro-ide_1
bb50c29a35c8        mayankt/webserver:b                    "/bin/sh -c 'nginx'"     13 seconds ago      Up 10 seconds       80/tcp, 443/tcp   
                                                                     nitroide_webserver-b_1
``` 

### Step 2

Once all your containers are running successfully, navigate to your IDE's web console. If you are following along on your local machine, go to url [http://localhost:9090](http://localhost:9090).

If you are following along in the sandbox environment, navigate your local browser to [http://userX-ide.sl.americasreadiness.com](http://userX-ide.sl.americasreadiness.com) where `X` denotes your user number in the FQDN. 

  >Please wait up to 2 minutes for the IDE and CPX to fully load before they are accessible via the web console. Usually services are available within 30 seconds of deployment. 

You should be greeted with Cloud9's loading page and then ultimately the IDE editor pane. Within the side pane you should notice your `workspace` directory and within that directory you should see the `nitro-ide` repository on your docker host. 

You can select any file to open and edit it or to examine it. You even have access to the container's CLI terminal in the bottom pane. In the container's CLI pane within Cloud9 IDE, enter the following commands: 

`git clone -b cpx-101 https://github.com/Citrix-TechSpecialist/NetScalerNITRO.git`

### Step 3

A new directory will have beeen created `NetScalerNITRO` with the `nsAuto.py` python script that is pre-coded to configure the CPX to loadbalance webserver-a and webserver-b. 

In the bottom pane within the container's CLI, enter the following commands to configure the CPX via NITRO scripted with Netscaler's Python SDK. Desired state configuration is specified in the `nsAutoCfg.json` file with pre-seeded default values for our environment (i.e. backend webserver IP's and CPX default username and pass along with its NSIP.)

  >It is highly encouraged to open the `nsAuto.py` and `nsAutoCfg.json` file within the IDE to examine and learn from its contents and understand how the script is coded with NetScaler's NITRO Python SDK. 

```
cd NetScalerNITRO
python nsAuto.py
```

You will see an output similar to: 

```
Configuring NS 
Starting to configure...
All done preforming configuration
```

This indicates that the CPX has been configured successfully. It is loadbalancing Webserver A and Webserver B on its port 10000 using it's docker container IP in the sandbox docker network. Container port 10000 is mapped to host port 1000 so you can access your load balancer at [http://localhost:10000](http://localhost:10000) if you are following along in the lab locally.

If you are following along in the sandbox environment, navigate your local browser to [http://userX-lb.sl.americasreadiness.com](http://userX-lb.sl.americasreadiness.com) where `X` denotes your user number in the FQDN. 

### Conclusion 



