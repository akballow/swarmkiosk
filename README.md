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


## Preparing the Raspberry Pi's
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
If you see chrome load, then everything is set. If you get display can not be found, you need to check that you applied 'xhost +local:root' on the Pi.


## Create the first container to visualize the containers:
On the Manager run the following command for the type of hardware
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

docker run -v /tmp/.X11-unix:/tmp/.X11-unix --memory 512mb -e DISPLAY=unix$DISPLAY icebob/chromium-armhf https://www.docker.com/
```
# credit
I wanted to give credit to https://github.com/icebob/docker-chromium-armhf for the dockerfile source which built thid program.

