---
layout:     post
title:      Easy Guide to Installing Docker
subtitle:   TL;DR copy paste the code ;)
date:       2024-02-02
author:     Shrijal Pradhan
header-img: "img/pha-easy-docker.png"
color-grad: 0.75
catalog: true
categories: blog
permalink: /:categories/easy-guide-to-installing-docker/
tags:
    - pha
    - docker
---

## Introduction

Docker is a platform for developing, shipping, and running applications in containers. Containers are lightweight, portable, and isolated environments that package an application along with its dependencies, ensuring consistent behavior across different environments. With Docker, you can easily create, deploy, and manage containers using Dockerfiles, Docker images, and Docker Compose. Docker simplifies the development workflow, improves collaboration between teams, and streamlines the deployment process by abstracting away infrastructure dependencies (*Powered by [ChatGPT][ChatGPT]*).

So let's build from the ground up. Here are the specifications for the base Operating System that we will use for our docker setup:
- Ubuntu 22.04
- NVIDIA Driver 525
- Docker version 24.0.7
- NVIDIA Docker 2.13.0

This is the bare minimum requirement to get started with the mentioned docker setup. The choice of the operating system as Ubuntu 22.04 was made for the longevitiy of the system. Additionally, it is important for modern robotics projects to be supported with GPU computations. To this end, we will enable Docker to use the GPU via NVIDIA GPU Drivers. The blog post from Roboflow regarding [Using GPU in Docker][Using GPU in Docker] explains the topic in a simple way. The [NVIDIA Container Toolkit][NVIDIA Container Toolkit] allows users to build and run GPU accelerated containers. The toolkit includes a container runtime library and utilities to automatically configure containers to leverage NVIDIA GPUs.

> **_NOTE:_**
> The Version of Docker and NVIDIA Docker should note that relevant, it is advised to use the latest available vesion.

## Installing Ubuntu

The two main concepts in installing Ubuntu is to [Create a bootable USB stick][Create a bootable USB stick] and [Dual Boot][Dual Boot]. As a basis for this guideline, install Ubuntu as per individual needs and resource availabilities. However, it is suggested to allocate atleast 150 GB to the `ROOT` Folder of Ubuntu as that is where Docker images and containers are stored by default.

As these are a set of instructions that cannot be narrowed down to a simple script (atleast for now), please refer to the referenced links.

> **_NOTE:_**
> It should also be possible to install Ubuntu directly in Windows via [Windows Subsystem for Linux][WSL2] and use docker directly inside it bit this scope will not be explored in this article.

## Installing NVIDIA Driver

> **_NOTE:_**
> A NVIDIA driver is a software program that enables communication between your computer and the NVIDIA graphics processor installed in your system. It is used to ensure that your hardware works as intended with the latest software, games and applications. (*[Citation][NVIDIA Drivers]*)

Check if the NVIDIA Driver are installed.

```bash
nvidia-smi
```
If the drivers are installed then this command would produce no errors and would show details regarding the GPU device(s) available. 

> **_WARN:_**
> Only follow the instructions below if the nvidia drivers are not installed. Improperly setting up the NVIDIA Drivers may cause the system not boot up.

```bash
sudo apt install nvidia-driver-525 -y
```

## Installing Docker

As this is an Open Source project, we will use only docker engine, which is free. Read the [Docker Overview][Docker Overview] to tingle the knowledge taste buds. 

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

Once Docker Engine is installed, we need to setup priviliges to run Docker without `root` as documented in [Docker Post-Install].

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

The user must then log out and back in for the settings to take effect. 

## Installing NVIDIA Docker

Now that we have installed Docker, we will configure the compatibility of Docker with NVIDIA Drivers. We do this with [NVIDIA Container Toolkit][Install NVIDIA Container Toolkit] which replaces [Nvidia-Docker][Nvidia-Docker]. The toolkit allows users to build and run GPU-accelerated containers. The article [Using GPU in Docker][Using GPU in Docker] goes into more depth in the matter.

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

Once these installations are completed, we can start with docker containers. With the acception of sensor and ROS drivers; everything else required for the project can be installed within the containers.

## Conclusion

This guideline is part of the [PHA Project]. The next article set's up a docker image for a robotics project [[Chaotic Docker]].

## Short Introduction to Docker (Bonus)

Here's a short guide to Docker (*Powered by [ChatGPT][ChatGPT]*):

1. **Understanding Containers**:
   - Containers are lightweight, portable, and self-sufficient environments that package an application along with its dependencies.
   - They enable consistent development, testing, and deployment across different environments.

2. **Installing Docker**:
   - Visit the official Docker website (https://www.docker.com/) and download Docker Desktop for your operating system.
   - Follow the installation instructions provided for your platform.

3. **Basic Docker Concepts**:
   - **Images**: Blueprints for containers, containing the application code, libraries, dependencies, and configurations.
   - **Containers**: Instances of Docker images that run applications in isolated environments.
   - **Dockerfile**: A text file that contains instructions to build Docker images.
   - **Docker Compose**: A tool for defining and running multi-container Docker applications.

4. **Creating Docker Images**:
   - Write a Dockerfile to specify the configuration and dependencies of your application.
   - Use the `docker build` command to build an image based on the Dockerfile.
   - Tag the image with a name and version using the `-t` flag.

5. **Running Docker Containers**:
   - Use the `docker run` command to create and start a container from an image.
   - Specify options such as port mappings, volumes, and environment variables as needed.
   - Manage running containers with commands like `docker ps`, `docker stop`, `docker start`, and `docker rm`.

6. **Networking and Volumes**:
   - Docker provides networking capabilities to enable communication between containers and the outside world.
   - Volumes allow sharing data between the host system and containers, or between containers.

7. **Docker Compose**:
   - Docker Compose uses YAML files to define multi-container applications.
   - It simplifies the process of managing complex applications with multiple services.

8. **Docker Hub**:
   - Docker Hub is a repository of Docker images that you can use to share and discover containerized applications.
   - You can also push your own Docker images to Docker Hub for distribution and collaboration.

9. **Best Practices**:
   - Keep your Docker images small by minimizing layers and removing unnecessary dependencies.
   - Regularly update your base images and dependencies to patch security vulnerabilities.
   - Use `.dockerignore` files to exclude unnecessary files and directories from your Docker images.

10. **Learning Resources**:
    - Docker Documentation: https://docs.docker.com/
    - Docker Official Images: https://hub.docker.com/
    - Dockerfile Best Practices: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

This guide should give you a solid foundation to start using Docker for your development and deployment needs. Dive deeper into Docker's documentation and community resources for more advanced topics and best practices.

## References

- [NVIDIA Drivers]
- [Docker Overview]
- [Install Docker Engine]
- [Docker Post-Install] 
- [Using GPU in Docker]
- [Install NVIDIA Container Toolkit]
- [Nvidia-Docker]
- [Rocker]
- [NVIDIA Container Toolkit]
- [Create a bootable USB stick]
- [Dual Boot]
- [WSL2]
- [ChatGPT]

[PHA Project]: {{site.url}}/pha-project/
[Chaotic Docker]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-docker-for-robotics-research/
[NVIDIA Drivers]: https://www.lenovo.com/us/en/glossary/nvidia-drivers/
[Docker Overview]: https://docs.docker.com/get-started/overview/
[Install Docker Engine]: https://docs.docker.com/engine/install/ubuntu/
[Docker Post-Install]: https://docs.docker.com/engine/install/linux-postinstall/ 
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Install NVIDIA Container Toolkit]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
[Nvidia-Docker]: https://github.com/NVIDIA/nvidia-docker
[Rocker]: https://github.com/osrf/rocker
[NVIDIA Container Toolkit]: https://github.com/NVIDIA/nvidia-container-toolkit
[Create a bootable USB stick]: https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview
[Dual Boot]: https://www.tecmint.com/install-ubuntu-alongside-with-windows-dual-boot/
[WSL2]: https://learn.microsoft.com/en-us/windows/wsl/about
[ChatGPT]: https://chat.openai.com/