# Raspberry Pi Docker Swarm Kiosk: swarmkiosk for armhf

## Requirements
1. Atleast 1 static IP for the docker manager, preferably two.
2. Raspberry Pi 3's or new model to have enough horsepower.

## Build
You can build your own docker image from the souce dockerfile or you can use the image kirilballow/chrome:latest
```
git clone https://github.com/akballow/swarmkiosk
cd swarmkiosk
docker build -t kirilballow/chrome .
```

## Preparing the static manager
This could be a server which can have a static ip set, or another raspberry pi which can have the static ip set. 
The manager needs to have a static ip because if the ip changes, the whole swarm will have issues.
I also do not recommend the Raspberry Pi to be the manager as the resources are already sparse.
Install Docker
```
curl -sSL get.docker.com | sh
```
Start the Swarm
```
docker swarm init
```
Get the join token for your workers
```
docker swarm join-token worker
```


## Preparing the Raspberry Pi's (Each Pi)
I recommend you to pull out that SD card from your pi and reformat it and install the latest fresh NOOBS software on it.
Install the Raspberry OS with gui (if you are new to this)
update the software
```
apt-get update
apt-get upgrade
```
Enable VNC and SSH
```
raspi-config 
```
Disable screen power management (`export DISPLAY=:0` on ssh or run in local terminal)
```
xset s noblank
xset s off
xset -dpms
```
On the host, you need to allow the docker user access to your local X session
```
xhost +local:root
echo "xhost +local:root" >> ~/.profile
```
Install Docker
```
curl -sSL get.docker.com | sh
```
Join the swarm using the output from the 'docker swarm join-token worker' from the manager
```
docker swarm join --token XXXXX XXX.XXX.XXX.XXX:2377
```
Test that chrome kiosk works on the local node
```
docker run -v /tmp/.X11-unix:/tmp/.X11-unix --memory 512mb \
-e DISPLAY=unix$DISPLAY kiril.ballow/chrome:lastest https://www.docker.com/
```
If you see chrome load with the website docker.com, then everything is set. If you get display :0 can not be found, you need to check that you applied 'xhost +local:root' on the Pi.

## Create the first container to visualize the containers
On the Manager run the following command for the type of hardware. Note that container images have to have the same hardware architecure to run. For example Pi's use armhf and servers are x86_64.
Manager that is not a Pi (other then armhf):
```
docker service create --name=viz --publish=8080:8080/tcp --constraint=node.role==manager \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock dockersamples/visualizer
```
Manager which is a Pi (armhf):
```
sudo docker service create --name viz-pi --publish 8080:8080/tcp \
--constraint node.role==manager --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
alexellis2/visualizer-arm
```
You can now visit the website <Manager-IP>:8080 to see the swarm with the single viz container. If you can not hit the IP, make sure firewall allows the port.

## Create the first broadcasted kiosk website
Lets make website docker.com load on all the Pi's at the same time.
```
docker service create --name=Website1 --replicas=0 --replicas-max-per-node=1 --constraint='node.role==worker' \
--mount type=bind,src=/tmp/.X11-unix,dst=/tmp/.X11-unix --env DISPLAY=:0 \
kirilballow/chrome https://docker.com
```
Quick explination of options
1. --name is the name of the service
2. --replicas is the number of copies of it. we start with 0, so we can enable it later.
3. --replicas-max-per-node is number of replicas we want on a node. We set this to 1 since we only need on per node. An alternive to --replicas and --replicas-max-per-node is to set the mode to global with --mode=global. The issue with global service is that you can not disable it, you can just set a constraint value to something that doesnt exsist. Basicly if you know what you are doing using global is better overall.
4. --constraint is set for only making this service run on workers, if the manager is a pi, we wouldnt want this set.
5. --mount, this one is tricker to understand. We want to bind local host path to the continer path for the x11 display server. this is how the docker continaer knows which local host display to use. 
6. --env we want to set the DISPLAY env to the local host display which is typically :0

Starting the service, get the number of Pi's and set the replicas to it. 
```
docker service update Website1 --replicas=<Number_Of_Pi's>
```

At this time you will hopefully see every Pi display the website now. You can even visit the <Manager-IP>:8080 on the webbrowser to see that each worker should have a continer spun up.
  
To switch the website, change the argument
```
docker service update Website1 --args="test.com"
```
  
To stop the service, set the replicas to 0 again.
```
docker service update Website1 --replicas=0
```

## Automate the websites cycle
You will want to make a shell script on the manager. An example of will be
```
#!/bin/bash
while drue; do
  docker service update Website1 --args="example1.com"
  sleep 120
  docker service update Website1 --args="example2.com"
  sleep 120
  docker service update Website1 --args="example3.com"
done
```

## Setting up a second manager
You will want to have a second manager in the case that the main manager goes down. With only one manager the swarm dies. If you set another server as a manager, you have HA in a way.
You can either promote a Raspberry pi from the manager
```
docker node ls
docker node promote <ID_from_last_command>
```
Or join the swarm with the manager token.
from the manager
```
docker swarm join-token manager
```
on the second to be manager with the output from above.
```
docker swarm join --token XXXXX XXX.XXX.XXX.XXX:2377
```

# credit
I wanted to give credit to https://github.com/icebob/docker-chromium-armhf for the dockerfile source which built thid program.

