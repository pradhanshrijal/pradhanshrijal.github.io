---
layout:     post
title:      A Chaotic Guide to Docker for Robotics Research
subtitle:   A quick and messy guide to docker on overdrive
date:       2024-01-19
author:     Shrijal Pradhan
ext-img: "https://miro.medium.com/v2/resize:fit:1001/1*ZB574gbdvY0YzIr1x83YTg.png"
catalog: true
categories: blog
tags:
    - pha
    - docker
    - robotics
    - ros
---

## Introduction

There are already several very insightful articles available which details the basics of docker and how it's features can be applied into the area of robotics. The [Docker ROS Guide][Docker ROS Guide] and it's update counterpart [Docker ROS 2 Guide][Docker ROS 2 Guide] provide good introductions for using Docker and ROS in cohesion. On the other hand, the [Ubuntu White Paper][Ubuntu White Paper] puts forth some arguments against the use of Docker and ROS. In that light, this article is not about the basics of Docker or ROS. 

This article delves into a new approach to using Docker. An approach that will supercharge Docker for a large robotics stack. We will list down all the necessary additional features when using Docker for robotics research, specifically with ROS, and provide substancial use cases and explanations for the said features. We also have to conform a stable driver base within the main operating system for the usage of the docker containers. 

## Setting up the base OS

So let's build from the ground up. Here are the specifications for the base Operating System that we will use for our docker setup:
- Ubuntu 22.04
- NVIDIA Driver 525
- Docker version 24.0.7
- NVIDIA Docker 2.13.0

This is the bare minimum requirement to get started with the mentioned docker setup. Other than these requirements, everything else can be installed and re-installed (inside the docker container of course). The choice of the operating system as Ubuntu 22.04 was made for the longevitiy of the system. Additionally, it is important for modern robotics projects to be supported with GPU computations. To this end, we will enable Docker to use the GPU via NVIDIA GPU Drivers. The blog post from Roboflow regarding [Using GPU in Docker][Using GPU in Docker] explains the topic in a simple way. The [NVIDIA Container Toolkit][NVIDIA Container Toolkit] allows users to build and run GPU accelerated containers. The toolkit includes a container runtime library and utilities to automatically configure containers to leverage NVIDIA GPUs.

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

> The Version of Docker and NVIDIA Docker should note that relevant, it is advised to use the latest available vesion.

#### Installing Ubuntu

The two main concepts in installing Ubuntu is to [Create a bootable USB stick][Create a bootable USB stick] and [Dual Boot][Dual Boot]. As a basis for this guideline, install Ubuntu as per individual needs and resource availabilities. However, it is suggested to allocate atleast 150 GB to the `ROOT` Folder of Ubuntu as that is where Docker images and containers are stored by default.

As these are a set of instructions that cannot be narrowed down to a simple script (atleast for now), please refer to the referenced links.

> It should also be possible to install Ubuntu directly in Windows via [Windows Subsystem for Linux][WSL2] and use docker directly inside it bit this scope will not be explored in this article.

#### Installing NVIDIA Driver

> A NVIDIA driver is a software program that enables communication between your computer and the NVIDIA graphics processor installed in your system. It is used to ensure that your hardware works as intended with the latest software, games and applications. (*[Citation][NVIDIA Drivers]*)

Check if the NVIDIA Driver are installed.

```bash
nvidia-smi
```
If the drivers are installed then this command would produce no errors and would show details regarding the GPU device(s) available. 

> Only follow the instructions below if the nvidia drivers are not installed. Improperly setting up the NVIDIA Drivers may cause the system not boot up.

```bash
sudo apt install nvidia-driver-525 -y
```

#### Installing Docker

As this is an Open Source project, we will use only docker engine, which is free. Read the [Docker overview][Docker overview] to tingle the knowledge taste buds. 

Based on [Install Docker Engine][Install Docker Engine]:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Once Docker Engine is installed, we need to setup priviliges to run Docker without `root` as documented in [Docker post-install].

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

The user must then log out and back in for the settings to take effect. 

#### Installing NVIDIA Docker

Now that we have installed Docker, we will configure the compatibility of Docker with NVIDIA Drivers. We do this with [NVIDIA Container Toolkit][Install NVIDIA Container Toolkit] which replaces [nvidia-docker][nvidia-docker]. The toolkit allows users to build and run GPU-accelerated containers. The article [Using GPU in Docker][Using GPU in Docker] goes into more depth in the matter.

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

<figure class="img-with-text">
    <img src="https://blog.roboflow.com/content/images/2020/05/image-39.png" />
    <figcaption>NVIDIA Container Toolkit (<a href="https://blog.roboflow.com/use-the-gpu-in-docker/">Citation</a>)</figcaption>
</figure>

Once these installations are completed, we can start with docker containers. Everything else required for the project can be installed within the containers.

## Single source of information

The main idea of this article is to create a single space for all the parameters and specifications for a robotics project. This idea helps to make sure that all the parameters required to setup a robot are referenced from and can be changed via a single file. This helps to maintain the complexity of a large robotics projects with several sub-modules. Further, these specifications generally includes weights and models that might not be be feasible to be used within a docker container. To this end, the single source of information must be place within a shared space between the docker and the main operating system. Such a space also is not allocated to the total usage space of docker when works at an advantage when the same weights have to be used in two container instances (say when comparing some changes to a module).

The remainder of the article would then focus on this concept and explain in detail each component that would allow docker to be used in such a manner.

## Sample Usage

Let's start with a sample usage of the concept, and then we can break it down piece by piece. For this purpose, we 

```bash
```

## Advantages

## Disadvantages

## Conclusion

This article creates a base platform for the [PHA Project] where different modules of the automated driving stack could be build upon.

## Possible Future Ideas

These are ideas that not in the current plan (atleast within the [PHA Project]) but were interesting ideas founding during the research of this article:
- Live Stick: The idea is to be able to stick a USB Stick and easily run a script (with the address of the stick of course) that installs everything required outside the docker container.

## References

- [Docker ROS Guide] - https://roboticseabass.com/2021/04/21/docker-and-ros/
- [Docker ROS 2 Guide] - https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
- [Ubuntu White Paper] - https://ubuntu.com/engage/dockerandros
- [NVIDIA Drivers] - https://www.lenovo.com/us/en/glossary/nvidia-drivers/
- [Docker overview] - https://docs.docker.com/get-started/overview/
- [Install Docker Engine] - https://docs.docker.com/engine/install/ubuntu/
- [Docker post-install] - https://docs.docker.com/engine/install/linux-postinstall/ 
- [Using GPU in Docker] - https://blog.roboflow.com/use-the-gpu-in-docker/
- [Install NVIDIA Container Toolkit] - https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
- [nvidia-docker] - https://github.com/NVIDIA/nvidia-docker
- [rocker] - https://github.com/osrf/rocker
- [NVIDIA Container Toolkit] - https://github.com/NVIDIA/nvidia-container-toolkit
- [Create a bootable USB stick] - https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview
- [Dual Boot] - https://www.tecmint.com/install-ubuntu-alongside-with-windows-dual-boot/
- [WSL2] - https://learn.microsoft.com/en-us/windows/wsl/about

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/
[NVIDIA Drivers]: https://www.lenovo.com/us/en/glossary/nvidia-drivers/
[Docker overview]: https://docs.docker.com/get-started/overview/
[Install Docker Engine]: https://docs.docker.com/engine/install/ubuntu/
[Docker post-install]: https://docs.docker.com/engine/install/linux-postinstall/ 
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Install NVIDIA Container Toolkit]: 
[nvidia-docker]: https://github.com/NVIDIA/nvidia-docker
[rocker]: https://github.com/osrf/rocker
[NVIDIA Container Toolkit]: https://github.com/NVIDIA/nvidia-container-toolkit
[Create a bootable USB stick]: https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview
[Dual Boot]: https://www.tecmint.com/install-ubuntu-alongside-with-windows-dual-boot/
[WSL2]: https://learn.microsoft.com/en-us/windows/wsl/about