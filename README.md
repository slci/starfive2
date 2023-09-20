# ROS2 for VisionFive 2

This repository contains documentation for installing ROS2 on a StarFive VisionFive v2 board running Ubuntu 23.04. As of September 2023, ROS2 is (still) not distributed as an apt package or docker container for RISC-V, so must either be cross compiled or built natively. This repository contains a base image of ROS2 build using docker buildx, which has been published to a public container repo. Feel free to use this as a base image for your own docker containers.

## VisionFive 2 Setup
  - [X] [VisionFive 2](https://www.starfivetech.com/en/site/boards)
  - [X] [Ubuntu 22.04 RISC-V image](https://cdimage.ubuntu.com/releases/23.04/release/ubuntu-23.04-preinstalled-server-riscv64+visionfive2.img.xz) for VisionFive 2
  - [X] [USB to Serial Converter](https://t.ly/4eLGK)
  - [X] Git
  - [X] Docker.io
  - [X] Docker-Compose

### SBC Setup
 1. Update SPL and U-Boot described in [3.8.1. Updating SPL and U-Boot of Flash](https://doc-en.rvspace.org/VisionFive2/PDF/VisionFive2_QSG.pdf) 
 1. Flash a [Ubuntu 23.04 image](https://cdimage.ubuntu.com/releases/23.04/release/ubuntu-23.04-preinstalled-server-riscv64+visionfive2.img.xz) to an SD Card e.g. using [Balena Etcher](https://www.balena.io/etcher)
 1. Boot the VisionFive in SPI flash mode (DIP switches RGPIO_0 = 0, RGPIO_1 = 0)
 1. Install some software:
    ``` bash
    sudo apt install git
    sudo apt install docker.io
    sudo apt install docker-compose
    ```
  1. Configure docker to not require sudo (optional)
      ```bash
      sudo gpasswd -a $USER docker
      ```



## ROS2 Docker Image
### Using Docker Directly
This method allows you to use the demo applications to validate your install; however deriving your own container from the released version is recommended.

```bash
docker run --network host -i -t slci/ros:iron /bin/bash
```

### Using as the root for your own container

Create a dockerfile for your image (example below):

``` docker
FROM slci/ros:iron

RUN apt-get update && \
  apt install \
  libopencv-dev \
  python3-opencv \
  python-is-python3 \
  libboost-all-dev \
  openssl \
  git \
  gdb \
  i2c-tools \
  libcurl4-openssl-dev \
  libssl-dev \
  curl \
  libi2c-dev


ENTRYPOINT ["./ros_entrypoint.sh"]
CMD [ "bash" ]
```

I recommend using a Docker-Compose file, so that you don't have to remember the docker command line to launch your containers.

``` yaml
version: '3.4'
services:
  mangopi:
    image: mangopiws
    network_mode: "host"
    privileged: true
    cap_add:
      - SYS_PTRACE
    security_opt:
      - seccomp:unconfined   
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    devices:
      - /dev:/dev
    build:
      context: .
      dockerfile: ./Dockerfile
    command: /bin/sh -c "while sleep 1000; do :; done"
```

you can then use the command line:

``` bash
docker-compose up 
```
then

``` bash
docker exec -ti <ros container> /bin/bash
```



## Resources
- [Ubunti Wiki RISC-V/StarFive VisionFive 2](https://wiki.ubuntu.com/RISC-V/StarFive%20VisionFive%202)
- [VisionFive 2 Single Board Computer Quick Start Guide](https://doc-en.rvspace.org/VisionFive2/PDF/VisionFive2_QSG.pdf)
