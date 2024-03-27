---
layout:     post
title:      A Chaotic Guide to Using Docker for Robotics Research - Part I
subtitle:   A detailed guide to docker on overdrive
date:       2024-02-15
author:     Shrijal Pradhan
ext-img: "https://oneclick-cloud.com/wp-content/uploads/2023/08/Bigstock_-139961875-Docker-Emblem.-A-Blue-Whale-With-Several-Containers.-e1574090673987-1.jpg"
catalog: true
categories: blog
permalink: /:categories/:title/
tags:
    - pha
    - docker
    - robotics
    - ros
---

## Introduction

There are already several very insightful articles available which details the basics of docker and how it's features can be applied into the area of robotics. The [Docker ROS Guide][Docker ROS Guide] and it's update counterpart [Docker ROS 2 Guide][Docker ROS 2 Guide] provide good introductions for using Docker and ROS in cohesion. On the other hand, the [Ubuntu White Paper][Ubuntu White Paper] puts forth some arguments against the use of Docker and ROS. In that light, this article is not about the basics of Docker or ROS. 

This article series delves into a new approach to using Docker. An approach that will supercharge Docker for a large robotics stacks. We will list down all the necessary additional features when using Docker for robotics research, specifically with ROS, and provide simple use cases and explanations for the said features. This article is the first of a three part series. 

> **_NOTE:_**
> Install Docker with [Easy Guide to Installing Docker].

## Single Source of Information

The main idea of this article is to create a Single Source of Information (SSI) for all the parameters and specifications for a robotics project. This idea helps to make sure that all the parameters required to setup a robot are referenced from and can be changed via a single file. This helps to maintain the complexity of a large robotics projects with several sub-modules. Further, these specifications generally includes weights and models that might not be be feasible to be used within a docker container. To this end, the single source of information must be place within a shared space between the docker and the main operating system. Such a space also is not allocated to the total usage space of docker when works at an advantage when the same weights have to be used in two container instances (say when comparing some changes to a module).

The remainder of the article would then focus on this concept and explain in detail each component that would allow docker to be used in such a manner. See [Single Source of Truth - SSOT] for more information.

## TL;DR

If you just want to use docker, ROS and the [PHA Project] in the simplest way possible goto [TLDR; Docker Compose] for a quick usage of Docker.

## Sample Container Usage

Let's start with a sample usage of the concept, and then we can meticulously break it down piece by piece. For this purpose, we will use [PHA 22 Micro] which is an image specialized for this project. The details of this container are as follows:
- Ubuntu 22.04
- Cuda 11.7.1
- CuDNN 8.5.0.96
- ROS 2 Humble

The command to run this image as a container is:
```bash
xhost +local:docker
docker run \ 
    -d \
    --name pha-22-micro \
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
    -it phaenvs/pha-22:micro \
    /bin/bash
```

#### Run Description

`xhost` is used to allow a docker container to share the screen.

See [Docker run] for a full list of instructions.

| Option &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Description |
| `docker run` | The docker run command runs a command in a new container, pulling the image if needed and starting the container |
| `-d` `--detach` | Run container in background and print container ID |
| `--name` | Assign a name to the container |
| `-e` `--env` | Set environment variables |
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
| `-v` `--volume` | Bind mount a volume |
|  | `/tmp/.X11-unix:/tmp/.X11-unix` Pass the hosts display enviroment variable |
|  | `/home/${USER}/schreibtisch/pha_docker_files/docker_share:` `/home/pha/docker_share` Share a work folder between the host and docker |
|  | `/media/${USER}:/media/pha` Pass all mounted devices from the host to docker |
| `--gpus` | GPU devices to add to the container ('all' to pass all GPUs) |
| `-i` `--interactive` | Keep STDIN open even if not attached |
| `-t` `--tty` | Allocate a pseudo-TTY |

Now that we have a short introduction to the commands used, let's break it down from the simplest to the most complex.

#### Simple Container Run

```bash
docker run phaenvs/pha-22:micro
```

This command follows the principle:

- `docker run`: Tells Docker to run a container.
- `phaenvs/pha-22:micro`: Specifies the image name `phaenvs/pha-22` and the tag `micro` which presumably contains the application you want to run.

This would start a container based on the `phaenvs/pha-22:micro` image. However, The container would terminate after the application finishes unless you specify additional options to keep it running (*Powered by [Gemini][Gemini]*).

#### Adding a Name to the Container

In the previous example the created container would get a random name assigned to it, to assign a specific name, we can use the `--name` option.

```bash
docker run --name pha-22-micro phaenvs/pha-22:micro
```

Remember:
- This container name needs to be unique among all running containers.
- Using a descriptive name specific to the container is recommended for better organization (*Powered by [Gemini][Gemini]*).

#### Detaching the Container

For the example that we have been running till now the container would generally terminate when it is exited. Another example of it would be to run the container for a single instance with `--rm` option, which automatically removes the container when it exits.

```bash
docker run --name pha-22-micro --rm phaenvs/pha-22:micro
```

But this is not what we want, we want the container to be running on the background and to stay dormant when exited or stopped. For this purpose we will use `-d` option to detach it and `-it` to keep STDIN one even if the container is not attached and allocate a pseudo-terminal to interract with the application inside the container.

```bash
docker run -d --name pha-22-micro -it phaenvs/pha-22:micro
```

#### Command

We can send a command that is executed when the docker container starts. As we are running a detached container, what we will do is send the command `/bin/bash` so that our pseudo-TTY with start a bash shell. It is possible to replace this with a particular command that can be run on the container (but we will not do that here).

```bash
docker run -d --name pha-22-micro -it phaenvs/pha-22:micro /bin/bash
```

#### Sharing the Network

What we will do now is to share the host network with docker. This allows ROS 2 to communicate with several containers. Further, any applications running in the local ip can be shared with the container clusters. This can be done with `--network host`.

```bash
docker run -d --name pha-22-micro --network host -it phaenvs/pha-22:micro /bin/bash
```

A safer alternative to this might be to create a dedicated docker network, see [Docker network] for instructions. The applied method allows the docker container to be accessed remotely as well, during development it can be a key positive as the system (or the robot) can be accessed from anywhere.

#### Enable GPU for the CUDA Container

For many robotics projects, using the computation from GPUs has become standard practice. This allows us to run several machine learning based funtionalities like object detection and further also gives us the possibility to speed up heavy computations. See [Why GPUs] for more information regarding more details on the matter.

What we wiil do here is set the environment variables: `NVIDIA_VISIBLE_DEVICES=all` to specify the GPUs to the Docker CLI (in this case all available), for this we also need to set `--runtime=nvidia`. Further, we also set `NVIDIA_DRIVER_CAPABILITIES=all` to control which driver libraries/binaries will be mounted inside the container (again, in this case all available). Generally setting `--gpus all` is an alternative to this method but as we will later build complex softwares on top of the accessed GPUs, for now it's a safer idea to use both commands.

P.S. The command is getting bigger so let's start writing it down in many lines for simplicity of understanding.

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --runtime=nvidia \
    --network host \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

[Specialized Docker] goes into more details regarding the possible configurations. To learn more about container runtimes, see [Container Runtime].

#### Priviliges

What we will do now is to take out the training wheels that docker has on. The `--privileged` flag gives all capabilities to the container, and it also lifts all the limitations enforced by the device cgroup controller. In other words, the container can then do almost everything that the host can do. This flag exists to allow special use-cases, like running Docker within Docker (*[Citation][Docker run]*). This would be necessary for the coming steps.

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --runtime=nvidia \
    --privileged \
    --network host \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

Important Disclaimer: Using the following command with both --privileged and /bin/bash exposes your system to significant security vulnerabilities. It is strongly recommended to avoid this approach unless absolutely necessary and in a controlled environment (*Powered by [Gemini][Gemini]*).

This is an important point made by the [Ubuntu White Paper] that many researchers use docker in this manner and it exposes the system to many vurnerabilities. Make no mistake here, this command now grants docker unrestricted access to system and jeopardizing its integrity. However, for the system that we are trying to run this is a necessity. Alternatives to such priviliges will be later explored in the article. The alternatives will not be practically implemented to the extend of this article. This is **exactly** why the title is written as such. The scope of this article is for **research** and it is chaotic because these priviliges are required to run a robotics stack purely inside docker. Further, this eases this for development work and that is the goal of this article. So with this command we are allowing docker access to the systems resources.

#### Display

The first access we will give docker is to use the host display. This would allow us to view and run many things from within the docker container. For this purpose, we will set the host display as an environment variable within the container with `-e DISPLAY=$DISPLAY`, map the `X11` from the host to the container with `-v /tmp/.X11-unix:/tmp/.X11-unix` and also set the QT environment variable with `--env=QT_X11_NO_MITSHM=1`.

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    -e DISPLAY=$DISPLAY \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --env=QT_X11_NO_MITSHM=1 \
    --runtime=nvidia \
    --privileged \
    --network host \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

What this does for us is to run a simulation from within the container. This also allows us to run QT based applications such as RVIZ from within the container. See [X11 Forwarding],[Container GUI] regarding short instructions on Docker GUI. For an example of QT based applications see [ROS Docker GUI].

#### Mounting

The `/media` folder in Ubuntu is used for mounting removable media devices. This means that when you insert an external hard drive, USB flash drive, SD card, or other similar device, it will typically appear as a subfolder within `/media`.

Here's a breakdown of how it works:

- Automatic Mounting: When you plug in a removable device, Ubuntu automatically mounts it in a subfolder within `/media`. The subfolder name is usually based on a combination of the device type and a unique identifier. For example, a USB flash drive might be mounted as `/media/username/MY_FLASH_DRIVE`.
- Permissions: By default, only the user who plugged in the device has read and write access to the mounted folder. This helps ensure security and prevents unauthorized access to your data.
- Unmounted Devices: When you eject a removable device, the corresponding subfolder within `/media` is unmounted, making the device inaccessible until you plug it back in.

Here are some additional things to know about the `/media` folder:

- Manual Mounting: You can also manually mount and unmount devices using the mount and umount commands in the terminal. However, it's generally recommended to use the graphical user interface (GUI) or the eject button to avoid potential issues.
- Customization: In some cases, you might see additional folders within `/media` that are not related to removable devices. These could be leftover mount points from previous installations or custom configurations.
- Network Mounts: Network-attached storage (NAS) devices or shared folders on other computers can also be mounted under `/media`. However, the process for mounting these might differ slightly.

Overall, the `/media` folder plays a crucial role in managing and accessing data on removable devices in Ubuntu. It provides a convenient and secure way to interact with external storage without needing to manually configure mount points each time (*Powered by [Gemini][Gemini]*).

We will now map the `/media` folder to the container.

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    -e DISPLAY=$DISPLAY \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --env=QT_X11_NO_MITSHM=1 \
    --runtime=nvidia \
    --privileged \
    --network host \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /media/${USER}:/media/pha \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

What this does for us is that now we can access and external storage device from within a docker container. On a large-scale robotics project, the is a need to store large volumes of data, either for monitoring or debugging purposes. It is a good idea to store this data in an external SSD. No a very practical level, if such data is stored in the host device, there maybe incidences (quite often actually) that the data explores and there is no space in the host device to conduct any further processing. Additionally, if possible large files such as weights or checkpoints used to run machine learning models should also be saved and access from an external device. This is a general good practice for any robotics project with our without docker. 

#### Hardware Drivers

The `/dev` directory in Ubuntu is a special directory that contains device files. These files represent hardware devices connected to your system, allowing you to interact with them using software programs.

Here's a breakdown of what you'll find in `/dev`:

- Device Files: These are not regular files that store data, but rather act as interfaces for communicating with hardware devices. They come in two main types:
  + Block devices: Represent devices that store data in blocks, like hard drives, SSDs, and USB flash drives. Examples include `/dev/sda` for the primary hard drive and `/dev/sdb` for a secondary drive.
  + Character devices: Represent devices that deal with a stream of data, like keyboards, mice, printers, and serial ports. Examples include `/dev/tty` for terminals and `/dev/lp0` for the first parallel printer port.
- Naming Conventions: The naming conventions for device files can be cryptic, but often provide some clues about the device type. For example, sd typically refers to SCSI disk drives, while tty indicates a terminal device.
- Permissions: Most device files require root privileges to access and modify. This helps ensure that only authorized users can interact with critical hardware components.

Important Points to Remember:

- Do not modify device files directly: Messing with device files can lead to unexpected behavior or even damage your system. It's generally recommended to interact with hardware through device drivers and user interfaces provided by the operating system.
- Use tools for managing devices: You can use system administration tools or graphical utilities to view information about devices, format storage devices, or mount partitions.

Understanding the `/dev` directory is not essential for everyday usage, but it provides valuable insight into how your system interacts with hardware components. If you are interested in learning more about specific devices or troubleshooting hardware issues, consulting the Linux documentation or seeking help from experienced users is recommended (*Powered by [Gemini][Gemini]*).

So now we map the `/dev` folder into the container.

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    -e DISPLAY=$DISPLAY \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --env=QT_X11_NO_MITSHM=1 \
    --runtime=nvidia \
    --privileged \
    --network host \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /media/${USER}:/media/pha \
    -v /dev:/dev \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

This is necessary for us to be able to use sensor drivers (hardware devices) from within the containers. It is a good point to admit that the statement from the first article [Easy Guide to Installing Docker] that says no other software other than the mentioned basis would have to be installed outside of the docker containers is a perfect world scenario that does not exist. In reality certain things would always need to be installed in the host. Any sensor driver would have to be installed on the host and the docker container. An example would be the [Realsense SDK][Realsense SDK Guide].

#### Shared Memory

`--shm-size` refers specifically to the size limit of the shared memory segment associated with the `/dev/shm` directory in Linux systems.

Here's a breakdown of its meaning and relevance:

- Shared Memory: As explained previously, `/dev/shm` functions as a shared memory segment, allowing multiple processes to access and modify the same data in memory.
- Size Limit: The `--shm-size` parameter defines the maximum amount of RAM that can be allocated for shared memory within `/dev/shm`.
- Units: The size can be specified in bytes (b), kilobytes (k), megabytes (m), or gigabytes (g).

Context:

- This parameter is often encountered when working with Docker containers. By default, Docker allocates a limited amount of shared memory (usually 64MB) to containers (*Powered by [Gemini][Gemini]*).

```bash
docker run \ 
    -d \
    --name pha-22-micro \
    -e DISPLAY=$DISPLAY \
    --env=NVIDIA_VISIBLE_DEVICES=all \
    --env=NVIDIA_DRIVER_CAPABILITIES=all  \
    --env=QT_X11_NO_MITSHM=1 \
    --runtime=nvidia \
    --privileged \
    --shm-size=16gb \
    --network host \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /media/${USER}:/media/pha \
    -v /dev:/dev \
    --gpus all \
    -it phaenvs/pha-22:micro \
    /bin/bash
```

We need to expand this variable for when running large robotics stacks on a single docker container. This should be adjusted based on the requiments and available resources of the user.

#### Finally, Space for the SSI

At this boy the reader might be wondering, this author just quotes everything from [Gemini], why am I even reading this? We are getting to that. It was important to clearly and precisely explaning everything that we needed to setup. Further, as there are security risks involved with the use of docker in this manner, one can never be too informed. So from now we can focus on using docker the way [PHA Project] intends.

The [SSI](#single-source-of-information) is simple another folder mapped to the docker container, this folder will be shared between the host and all docker containers. This is where all the docker containers will access the relevant information from. This way if there are any changes to the project (as is the case when developing new things) the changes can be applied in one location and all the relevant modules are notified of it.

For this purpose we have to organise this folder in a particular manner so that all the containers know exactly where to find it. We can do this with the [PHA Docker]. The simplest explanation is that this is a space for all the automated scripts and parameter files relevant for the project.

All you have to do is make sure the `USER` variable is defined in the system.

```bash
echo $USER
```

If this is running then we can now setup the folder:

> **_NOTE:_**
> For users with the language setting in German, `Schreibtisch` and `schreibtisch` are two different folders.

```bash
cd /home/${USER}
mkdir schreibtisch
cd schreibtisch
git clone https://github.com/pradhanshrijal/pha_docker_files
```

Well that's about it, now we can repeat the `docker run` one final time, the one that we will use:

```bash
xhost +local:docker
docker run \ 
    -d \
    --name pha-22-micro \
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
    -it phaenvs/pha-22:micro \
    /bin/bash
```

The most important thing to note is that the folder `docker_share` is shared between the host and the docker containers. In the scope of this article, the folder is explicity ported to `/home/pha/docker_share` inside the docker container.

## Structure of the SSI

We can now disect the structure of the [SSI](#single-source-of-information). As we discussed earlier, the simplest form of SSI is just a folder that is mapped from the host to the containers. Now we will introduce some complexity into it to provide some struture to it and more stability when facing an even expanding project. This complexity (again) is just some folder spaces. Users are encouraged to create separate folders with clearly defining names when using the SSI. You can always create a new folder and write a short definition somewhere but if you have all the files in one place it will for sure get very confusing very soon. So within `docker_share` we have the following folders:

#### files

This is an empty folder that comes with PHA. This space is for model weights and other large files that cannot be part of a version control system. Users can freely decide on shaping this folder based on the project.

#### git_pkgs

This folder is where the user should clone all the git repositories. At this point the article is actually making an assumption. The assumption is that the user has organised the project into a kind of versioning control system like git that makes setting up the project easier. It is even better if the project is separted into several git repositories for modularity. If this is not the case then no worries! This space is for the functionalities in general, you can install all the functionalities here.

#### scripts

This folder is a place holder for all the automated scripts that are relevant for the project. It contains sub-folders to further separate the scripts.

**autostart:** This folder would include scripts for the supervisor to autostart scripts when the docker container is started.

**functions:** Any scripts that initiates a particular function would be stored here.

**install:** Scripts required to install additional packages into the container.

**setup:** This folder includes all files that are relevant for setting up the project. This includes installation scripts and start scripts.

**vars:** Any variables that needs to be declared to run a particular package can be written in a file and stored in this folder.

Users are encouraged to make any installations via a script so that they can be easily repeatable.

## Docker after running

With this method we are running docker in the background, now we will look at some additional functions:

#### Entering Docker

```bash
docker exec -it pha-22-micro /bin/bash
```

#### Stopping Docker

Don't forget to exit the container if you are inside of it.

```bash
docker stop pha-22-micro
```

#### Start Docker

Once you have initialized the container with `docker run` you don't have to run the command over and over again, We are storing it for future use:

```bash
docker start pha-22-micro
```

## Conclusion

This article explains containerizing docker with a Single Source of Information (SSI) which serves as a base to fine tune the parameters for a robotics project. An extensive example of SSI will be introduced on a later article. This article is part of the [PHA Project] where different modules of the automated driving stack could be build upon. The next article set's up a docker image for a robotics project [[Chaotic Docker - Part II]].

## Bibliography

- [Docker ROS Guide]
- [Docker ROS 2 Guide]
- [Docker run]
- [Docker network]
- [Ubuntu White Paper]
- [Single Source of Truth - SSOT]
- [PHA 22 Micro]
- [PHA Docker]
- [Using GPU in Docker]
- [Container GUI]
- [Docker GUI]
- [X11 Forwarding]
- [ROS Docker GUI]
- [Specialized Docker]
- [Container Runtime]
- [Gemini]
- [Why GPUs]
- [Realsense SDK Guide]
- [Shared Memory]

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Docker run]: https://docs.docker.com/reference/cli/docker/container/run/
[Docker network]: https://docs.docker.com/reference/cli/docker/network/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[Single Source of Truth - SSOT]: https://www.mulesoft.com/resources/esb/what-is-single-source-of-truth-ssot
[PHA Project]: {{site.url}}/pha-project/
[Chaotic Docker - Part II]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-ii/
[TLDR; Docker Compose]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-iii/#tldr-docker-compose
[PHA 22 Micro]: https://hub.docker.com/layers/phaenvs/pha-22/micro/images/sha256-42478c1cc914bc0255942bb4a2d27fbd2c2c6f1fc1d81771fb5a41ea6ebc08ce?context=explore
[PHA Docker]: https://github.com/pradhanshrijal/pha_docker_files 
[Easy Guide to Installing Docker]: {{site.url}}/{{page.categories}}/easy-guide-to-installing-docker/
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Container GUI]: https://leimao.github.io/blog/Docker-Container-GUI-Display/
[Docker GUI]: https://linuxmeerkat.wordpress.com/2014/10/17/running-a-gui-application-in-a-docker-container/
[X11 Forwarding]: https://forums.docker.com/t/x11-forwarding-with-v-on-docker-run-not-working/17708
[ROS Docker GUI]: https://wiki.ros.org/docker/Tutorials/GUI
[Specialized Docker]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
[Container Runtime]: https://developer.nvidia.com/container-runtime
[Gemini]: https://gemini.google.com/
[Why GPUs]: https://blogs.nvidia.com/blog/why-gpus-are-great-for-ai/
[Realsense SDK Guide]: https://dev.intelrealsense.com/docs/compiling-librealsense-for-linux-ubuntu-guide
[Shared Memory]: https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html