# swarmkiosk for armhf

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
```
curl -sSL get.docker.com | sh`
docker swarm init
docker swarm join-token worker
docker swarm join-token manager
```


## Preparing the Raspberry Pi
I recommend you to pull out that SD card from your pi and reformat it and install the latest fresh NOOBS software on it.
Install the full Raspberry OS and apt-get update, apt-get upgrade it.
```
raspi-config \\ Enable SSH and VNC
curl -sSL get.docker.com | sh
docker swarm join --token XXXXX XXX.XXX.XXX.XXX:2377 \\ You got this from the docker swarm join-token command
```

## Running
On the host, you need to allow the docker user access to your local X session
```
xhost +local:docker
```
If `xhost` is not found, install the `x11-xserver-utils` package.

## Run the container:
```
docker run -v /tmp/.X11-unix:/tmp/.X11-unix --memory 512mb -e DISPLAY=unix$DISPLAY icebob/chromium-armhf https://www.docker.com/
```

# credit
I wanted to give credit to https://github.com/icebob/docker-chromium-armhf for the dockerfile source which built thid program.

