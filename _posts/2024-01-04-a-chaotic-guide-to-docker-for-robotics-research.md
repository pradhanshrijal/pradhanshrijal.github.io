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

There are already several very insightful articles available which details the basics of docker and how it's features can be applied into the area of robotics. The [Docker ROS Guide][Docker ROS Guide] and it's update counterpart [Docker ROS 2 Guide][Docker ROS 2 Guide] provide a good introduction to using Docker and ROS in cohesion. On the other hand, the [Ubuntu White Paper][Ubuntu White Paper] puts forth some arguments against the use of Docker and ROS. In that light, this article is not about the basics of Docker or ROS. 

This article delves into a new approach to using Docker. An approach that will supercharge Docker for a large robotics stack. We will list down all the necessary additional features when using Docker for robotics research, specifically with ROS, and provide substancial use cases and explanations for the said features. We also have to conform a stable driver base within th main operating system for the usage of the docker containers.

## Setting up the base OS

So let's build from the ground up. Here are the specifications for the base Operating System that we will use for our docker setup:
- Ubuntu 22.04
- NVIDIA Driver 525
- Docker version 24.0.7
- NVIDIA Docker 2.13.0

This is the bare minimum requirement to get started with the mentioned docker setup. Other than these requirements, everything else can be installed and re-installed (inside the docker container of course). The choice of the operating system as Ubuntu 22.04 was made for the longevitiy of the system. Additionally, it is important for modern robotics projects to be supported with GPU computations. To this end, we will enable Docker to use the GPU via NVIDIA GPU Drivers. The blog post from Roboflow regarding [Using GPU in Docker][Using GPU in Docker] explains the topic in a simple way. The [NVIDIA Container Toolkit][NVIDIA Container Toolkit] allows users to build and run GPU accelerated containers. The toolkit includes a container runtime library and utilities to automatically configure containers to leverage NVIDIA GPUs.

> [!NOTE]
> The Version of Docker and NVIDIA Docker should note that relevant, it is advised to use the latest available vesion.

#### Installing Ubuntu

The two main concepts in installing Ubuntu is to [Create a bootable USB stick][Create a bootable USB stick] and [Dual Boot][Dual Boot]. As a basis for this guideline, install Ubuntu as per individual needs and resource availabilities. However, make sure to allocate atleast 150 GB to the `ROOT` Folder of Ubuntu as that is where Docker images and containers are stored by default.

> [!NOTE]
> It should also be possible to install Ubuntu directly in Windows via [Windows Subsystem for Linux][WSL2] and use docker directly inside it bit this scope will not be explored in this article.

#### Installing NVIDIA Driver

#### Installing Docker

<figure class="img-with-text">
    <img src="https://blog.roboflow.com/content/images/2020/05/image-39.png" />
    <figcaption>NVIDIA Container Toolkit (<a href="https://blog.roboflow.com/use-the-gpu-in-docker/">Citation</a>)</figcaption>
</figure>

#### Installing NVIDIA Docker

## Conclusion

This article creates a base platform for the [PHA Project][PHA Project] where different modules of the automated driving stack could be build upon.

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Install NVIDIA Container Toolkit Apt]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installing-with-apt
[NVIDIA Container Toolkit]: https://github.com/NVIDIA/nvidia-container-toolkit
[Create a bootable USB stick]: https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview
[Dual Boot]: https://www.tecmint.com/install-ubuntu-alongside-with-windows-dual-boot/
[WSL2]: https://learn.microsoft.com/en-us/windows/wsl/about