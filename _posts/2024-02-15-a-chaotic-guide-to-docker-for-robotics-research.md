---
layout:     post
title:      A Chaotic Guide to Docker for Robotics Research
subtitle:   A quick and messy guide to docker on overdrive
date:       2024-02-15
author:     Shrijal Pradhan
ext-img: "https://oneclick-cloud.com/wp-content/uploads/2023/08/Bigstock_-139961875-Docker-Emblem.-A-Blue-Whale-With-Several-Containers.-e1574090673987-1.jpg"
catalog: true
categories: blog
permalink: /:categories/a-chaotic-guide-to-docker-for-robotics-research/
tags:
    - pha
    - docker
    - robotics
    - ros
---

## Introduction

There are already several very insightful articles available which details the basics of docker and how it's features can be applied into the area of robotics. The [Docker ROS Guide][Docker ROS Guide] and it's update counterpart [Docker ROS 2 Guide][Docker ROS 2 Guide] provide good introductions for using Docker and ROS in cohesion. On the other hand, the [Ubuntu White Paper][Ubuntu White Paper] puts forth some arguments against the use of Docker and ROS. In that light, this article is not about the basics of Docker or ROS. 

This article delves into a new approach to using Docker. An approach that will supercharge Docker for a large robotics stack. We will list down all the necessary additional features when using Docker for robotics research, specifically with ROS, and provide substancial use cases and explanations for the said features. We also have to conform a stable driver base within the main operating system for the usage of the docker containers. 

> **_NOTE:_**
> Install Docker with [Easy Guide to Installing Docker].

## Single source of information

The main idea of this article is to create a single space for all the parameters and specifications for a robotics project. This idea helps to make sure that all the parameters required to setup a robot are referenced from and can be changed via a single file. This helps to maintain the complexity of a large robotics projects with several sub-modules. Further, these specifications generally includes weights and models that might not be be feasible to be used within a docker container. To this end, the single source of information must be place within a shared space between the docker and the main operating system. Such a space also is not allocated to the total usage space of docker when works at an advantage when the same weights have to be used in two container instances (say when comparing some changes to a module).

The remainder of the article would then focus on this concept and explain in detail each component that would allow docker to be used in such a manner.

## Sample Usage

Let's start with a sample usage of the concept, and then we can break it down piece by piece. For this purpose, we will use [PHA 22 Mini] which is an image specialized for this project. The details of this container are as follows:
- Ubuntu 22.04
- Cuda 11.7.1
- CuDNN 8.5.0.96
- ROS 2 Humble

The command to run this image as a container is:
```bash
xhost +local:docker
docker run \ 
    -d \
    --name pha-22-mini \
    -e DISPLAY=$DISPLAY \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --env=QT_X11_NO_MITSHM=1 \
    --runtime=nvidia \
    --privileged \
    --shm-size=16gb \
    --network host \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /home/${USER}/schreibtisch/pha_docker_files/docker_share:/home/pha/docker_share \
    -v /media/${USER}:/media/pha \
    -v /dev:/dev \
    --gpus all \
    -it phaenvs/pha-22-mini \
    /bin/bash
```

#### Description

`xhost` is used to allow a docker container to share the screen.

See [Docker run] for a full list of instructions.

| Option &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Description |
| `docker run` | The docker run command runs a command in a new container, pulling the image if needed and starting the container |
| `-d, --detach` | Run container in background and print container ID |
| `--name` | Assign a name to the container |
| `-e, --env` | Set environment variables |
| | `DISPLAY=$DISPLAY` Pass the Display of the base OS to docker |
| | `NVIDIA_VISIBLE_DEVICES=all` All GPUs will be accessible, this is the default value in base CUDA container images |
| | `NVIDIA_DRIVER_CAPABILITIES=all` Enable all available driver capabilities |
| | `QT_X11_NO_MITSHM=1` Allow QT applications to use the screen |
| `--runtime` | Runtime to use for this container |
| | `nvidia` register a new runtime during the creation of the container to expose NVIDIA GPUs |
| `--privileged` | Give extended privileges to this container |
| `--shm-size` | Size of /dev/shm |
| `--network` | Connect a container to a network |
|  | `host` Pass host network to docker container, also useful for swarm service |
| `-v, --volume` | Bind mount a volume |
| `--gpus` | GPU devices to add to the container ('all' to pass all GPUs) |
| `-i, --interactive` | Keep STDIN open even if not attached |
| `-t, --tty` | Allocate a pseudo-TTY |

## Advantages

## Disadvantages

## Conclusion

This article creates a base platform for the [PHA Project] where different modules of the automated driving stack could be build upon.

## References

- [Docker ROS Guide]
- [Docker ROS 2 Guide]
- [Docker run]
- [Ubuntu White Paper]
- [PHA 22 Mini]
- [Using GPU in Docker]
- [Container GUI]
- [Docker GUI]
- [Specialized Docker]
- [Container Runtime]

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Docker run]: https://docs.docker.com/reference/cli/docker/container/run/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/
[PHA 22 Mini]: https://hub.docker.com/r/phaenvs/pha-22-mini
[Easy Guide to Installing Docker]: {{site.url}}/blog/easy-guide-to-installing-docker/
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Container GUI]: https://leimao.github.io/blog/Docker-Container-GUI-Display/
[Docker GUI]: https://linuxmeerkat.wordpress.com/2014/10/17/running-a-gui-application-in-a-docker-container/
[Specialized Docker]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
[Container Runtime]: https://developer.nvidia.com/container-runtime