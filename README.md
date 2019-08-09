# swarmkiosk for armhf

## Build

```
git clone https://github.com/icebob/docker-chromium-armhf
cd docker-chromium-armhf
docker build -t icebob/chromium-armhf .
```

## Preparing the Raspberry Pi


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

