---
layout:     post
title:      A Chaotic Guide to Using Docker for Robotics Research - Part II
subtitle:   Creating a docker image, the PHA Way
date:       2024-02-28
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

In this article we will learn how to create custom docker images for robotics, the PHA way. For this purpose [PHA Git] has a second folder named `envs`. This folder already contains some sample projects, each identified by a folder. We will understand in detail how [PHA 22 Mini] was created. The `Dockerfile` is available in [PHA Git] inside `envs/pha22-mini`.

---
## Necessity

So up until this point we have learned how to run a container with this method and that should have been it for the article. But, we still have to talk about some specialized features [PHA 22 Mini] has compared to other docker container. The most important being the fact that rather than running the system as root, the container runs as a user. This is an important step for us because if you make any changes as root, it would overwrite the permissions within the shared folder. This means that even for the host the ownership of the changed file or folder would pass to root. Generally this is not what we want in our system.

---
## Sample Dockerfile

Here is what we will do to re-create [PHA 22 Mini]:

#### Set the Arguments and Environment Variables

As most of PHA is a project invovling a GPU, we have to select a GPU enabled docker image as the base. [PHA 22 Mini] is build upon `nvidia/cuda:11.7.1-devel-ubuntu22.04`. In addition, we have to select a version of [CuDNN]. A good way to check CuDNN compatibility with the selected image is by checking the details of the distribution in [CUDA Gitlab]. Let's have a look at a set of parameters that defines the details of the image:

```dockerfile
ARG IMAGE_NAME=nvidia/cuda
ARG IMAGE_VERSION=11.7.1-devel-ubuntu22.04
ARG DEBIAN_FRONTEND=noninteractive

FROM ${IMAGE_NAME}:${IMAGE_VERSION} as base
FROM base as base-amd64

ENV NV_USERNAME=pha
ENV NV_CUDA_VERSION_NUMBER=11.7
ENV NV_UBUNTU_VERSION=2204
ENV NV_CUDNN_VERSION 8.5.0.96
ENV NV_CUDNN_PACKAGE_NAME libcudnn8

ENV NV_SCRIPTS_PATH=/home/${NV_USERNAME}/docker_share/scripts
ENV NV_SOFTWARES_PATH=/home/${NV_USERNAME}/Softwares

ENV NV_ROS_VERSION=humble
```

For our running example, the details are as follows:

| **Platform** | **Version** |
| Ubuntu | 22.04 |
| CUDA | 11.7.1 |
| CuDNN | 8.5.0.96 |
| ROS | Humble |

We define folders within the image where we will save the scripts that we will use for installations and where we will install local softwares.

#### Time Zone

To install some softwares such as ROS, we need to declare a time-zone, this is done with the following command:

```dockerfile
# Declare Time Zone
ENV TZ=Europe/Berlin
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

#### CUDA Environments

Now we will define the CUDA variables, these environment variable only work with defining the main variable as it appears in the installation format.

```dockerfile
# Declare CUDA Environments
ENV NV_CUDA_VERSION=cuda${NV_CUDA_VERSION_NUMBER}
ENV NV_CUDA_FOLDER=cuda-${NV_CUDA_VERSION_NUMBER}

ENV NV_CUDNN_PACKAGE_VERSION ${NV_CUDNN_VERSION}-1

ENV NV_CUDNN_PACKAGE ${NV_CUDNN_PACKAGE_NAME}=${NV_CUDNN_PACKAGE_VERSION}+${NV_CUDA_VERSION}
ENV NV_CUDNN_PACKAGE_DEV ${NV_CUDNN_PACKAGE_NAME}-dev=${NV_CUDNN_PACKAGE_VERSION}+${NV_CUDA_VERSION}
```

#### Install General Packages

The CUDA Images do not have the CuDNN packages pre-installed. Here we install these required packages and hold them to this version so that they are not upgraded in the future. Generally, upgrades are an headache when considering docker and CuDNN as there may be some ML packages that don't work with the new upgrade. Further, here we also install some basic packages: `sudo`, `wget` and `vim`.

```dockerfile
# Update and install basics
RUN apt update && apt upgrade -y
RUN apt install sudo -y
RUN sudo apt install --no-install-recommends wget -y

RUN sudo apt install -y --no-install-recommends ${NV_CUDNN_PACKAGE} ${NV_CUDNN_PACKAGE_DEV}

RUN sudo apt-mark hold ${NV_CUDNN_PACKAGE_NAME}
RUN rm -rf /var/lib/apt/lists/*

# Install vim editor
RUN sudo apt update
RUN sudo apt install vim -y
```

#### User Setup

For the [PHA Project] what we have been doing is taking generally available docker instructions and merely organising it in such a way as to get the most out of it. Creating a user does a few things for us. First thing is that when accessing certain drivers as explained in [Chaotic Docker - Part I - Hardware Drivers], it makes a difference when you install the driver and grand access as root and as a sudo user. Basically, if you install packages as the root user, they cannot be accessed by the sudo user. This also holds true for installing python.

What we do here is pretty standard, we create a new user with the variable we define earlier using `useradd` and give sudo access to the user by adding it to the `sudo` group with `usermod`. As we are creating the first user as root, the `/home/${USER}` is owned by the root. We pass the ownership to the `${USER}` with `chown`. Then we change the user shell to `/bin/bash` with `chsh`. 

Once the user is setup, we can create a folder within the user directory which we will map from the host to the container as our work folder. For the [PHA Project] this is hard coded to `/home/${USER}/docker_share`. We already need this folder during image creation as most of the installations are done with scripts as we will see in the next sections.

```dockerfile
# Setup user
SHELL ["/bin/bash", "-c"]
RUN ["/bin/bash", "-c", "sudo useradd -m ${NV_USERNAME}"]
RUN ["/bin/bash", "-c", "sudo usermod -aG sudo ${NV_USERNAME}"]

WORKDIR /home/
RUN ["/bin/bash", "-c", "sudo chown -R ${NV_USERNAME}:${NV_USERNAME} /home/${NV_USERNAME}"]
WORKDIR /home/${NV_USERNAME}
RUN ["/bin/bash", "-c", "sudo chsh -s /bin/bash ${NV_USERNAME}"]
RUN mkdir /home/${NV_USERNAME}/docker_share
RUN mkdir ${NV_SCRIPTS_PATH}
```

#### Copy Scripts

From the Single Source of Information (SSI) Structure explained in [Chaotic Docker - Part I - SSI Structure], we only need to copy `setup` and `install` for image creation. This same copied space would be used as SSI by a container. Once these files are copied, we will also copy the `.bashrc` from the `root` home to the `user` home and like before change the ownership of the file. The script `term_disp.sh` shortens the full path of the actual folder to just the name of the actual folder that is displayed in the terminal. As PHA creates several nested folders this become necessary. With large projects with dozens of modules, this is the same case. After that we all allowing `sudo` to be used without password input, something that would be necessary for automated image generation (else someone might have to babysit the image as it is being generated).

```dockerfile
## Copy Scripts
RUN mkdir ${NV_SCRIPTS_PATH}/setup
COPY docker_share/scripts/setup ${NV_SCRIPTS_PATH}/setup
RUN mkdir ${NV_SCRIPTS_PATH}/install
COPY docker_share/scripts/install ${NV_SCRIPTS_PATH}/install

RUN sudo cp /root/.bashrc /home/${NV_USERNAME}/.
RUN echo -e "\n# Setup" >> /home/${NV_USERNAME}/.bashrc
RUN echo "export USER=${NV_USERNAME}" >> /home/${NV_USERNAME}/.bashrc
RUN echo "source ${NV_SCRIPTS_PATH}/setup/term_disp.sh" >> /root/.bashrc
RUN echo "source ${NV_SCRIPTS_PATH}/setup/term_disp.sh" >> /home/${NV_USERNAME}/.bashrc
RUN sudo chown -R ${NV_USERNAME}:${NV_USERNAME} /home/${NV_USERNAME}/.bashrc
RUN echo "${NV_USERNAME} ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers
```

#### Switch User

Pretty basic, we switch to the designated user.

```dockerfile
## Switch to User
RUN ["/bin/bash", "-c", "source /root/.bashrc"]
USER ${NV_USERNAME}
```

#### Install Base Packages

Now we can start with the installations of all the base packages used in the [PHA Mini]. First we set a path for the softwares that need to be installed locally. Before local installations, lets install python as debian packages. It is very important to perform this step after switching the user. Alongside `python3` we will also install `python3-pyqt5` for visualizations. We also install `pip` as it is a vital package installer and `venv` which is important for separating package installations specially when it comes to machine learning based packages. In addition we will also install the latest `cmake`, `g++` and `make`.

We will also include the sourcing of the CUDA paths to `.bashrc` so that that the CUDA Drivers can be found by other softwares.

```dockerfile
RUN mkdir ${NV_SOFTWARES_PATH}

# Setup Python
RUN sudo apt update
RUN sudo apt install python3 python3-pip \
                        python3-pyqt5 -y
#RUN python2 -m pip install --upgrade pip
RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install virtualenv

# Install based on scripts
WORKDIR "${NV_SOFTWARES_PATH}"
## CUDA Paths
RUN echo "source ${NV_SCRIPTS_PATH}/setup/cuda_paths.sh ${NV_CUDA_FOLDER}" >> /home/${NV_USERNAME}/.bashrc 
##

# Install apt packages
RUN sudo apt install tmux nautilus -y

## Upgrade Cmake
RUN sudo apt install cmake g++ make -y
##
```

Sample of `cuda_paths.sh`, keep in mind that the script is compatible with multiple CUDA Versions:

```bash
#!/bin/bash

# First input [$1] is the Cuda Folder - ex: cuda-11.3

PATH=/usr/local/$1/bin/:$PATH
CUDA_PATH=/usr/local/$1
CUDA_HOME=/usr/local/$1
LD_LIBRARY_PATH=/usr/local/$1/lib64:$LD_LIBRARY_PATH
CUDA_TOOLKIT_ROOT_DIR=/usr/local/$1
CUDACXX=/usr/local/$1/bin/nvcc
CUDA_INCLUDE_DIRS=/usr/local/$1/include
CUDA_CUDART_LIBRARY=/usr/local/$1/lib64/libcudart.so
CUDA_MODULE_LOADING=LAZY
```

#### Install ROS

This is the main package we need for robotics. [ROS Humble] should be a good starting point for anyone interested in this robotics middleware. In this article we will only cover how to install it. In the `Dockerfile` we do it via a script.

```dockerfile
RUN ["/bin/bash", "-c", "source /home/${NV_USERNAME}/.bashrc"]
RUN ["/bin/bash", "-c", "source ${NV_SCRIPTS_PATH}/install/install_ros_mini.sh ${NV_USERNAME} ${NV_ROS_VERSION}"]
```

Now we can look at the install script in more detail:

```bash
#!/bin/bash

# Installation for ROS 2 Minimum Version
# Ex: source install_ros_mini.sh pha humble

IN_USERNAME=$1
IN_ROS_VERSION=$2
IN_USERNAME="${IN_USERNAME:=pha}"
IN_ROS_VERSION="${IN_ROS_VERSION:=humble}"
source /home/${IN_USERNAME}/.bashrc

sudo apt install software-properties-common -y
sudo add-apt-repository universe -y

sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt upgrade -y

sudo apt install ros-$IN_ROS_VERSION-ros-base -y

sudo apt install ros-dev-tools -y

sudo apt install python3-colcon-common-extensions -y

echo -e "\n# ROS 2" >> /home/${IN_USERNAME}/.bashrc
echo "source /opt/ros/$IN_ROS_VERSION/setup.bash" >> /home/${IN_USERNAME}/.bashrc
source /opt/ros/$IN_ROS_VERSION/setup.bash
mkdir -p /home/${IN_USERNAME}/ros2_ws/src
cd /home/${IN_USERNAME}/ros2_ws
sudo rosdep init
rosdep update

cd src
mkdir tutorials
cd tutorials
git clone https://github.com/ros2/examples -b $IN_ROS_VERSION
cd /home/${IN_USERNAME}/ros2_ws
rosdep install -y --from-paths src --ignore-src --rosdistro $IN_ROS_VERSION # $ROS_DISTRO
colcon build --symlink-install
echo "source /home/${IN_USERNAME}/ros2_ws/install/setup.bash" >> /home/${IN_USERNAME}/.bashrc
echo -e "\n# Colon" >> /home/${IN_USERNAME}/.bashrc
echo "source /usr/share/colcon_cd/function/colcon_cd.sh" >> /home/${IN_USERNAME}/.bashrc
echo "export _colcon_cd_root=/opt/ros/$IN_ROS_VERSION/" >> /home/${IN_USERNAME}/.bashrc
echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> /home/${IN_USERNAME}/.bashrc
cd /home/${IN_USERNAME}
source /home/${IN_USERNAME}/.bashrc
```

This script automates the installation of ROS 2 Packages, for now it has been tested with `Foxy` and `Humble`. After it installs the requirements, it only installs the base ROS packages, this is done to keep the size of the corresponding image small. It further creates a sample ROS workspace and it also attaches `colcon_cd` to the system. See [Colcon Quick Directory] for more details.

#### Change Work Directory

The final step is to set the work directory to the home of the created user. Keep in mind that PHA does not explicity specify an entrypoint.

```dockerfile
WORKDIR "/home/${NV_USERNAME}"
```

---
## Create a Docker Image

If you have not created the folder structure as explained in [Chaotic Docker - Part I - Finally, Space for the SSI], setup the folder structure before we create the container.

```bash
cd /home/${USER}
mkdir schreibtisch
cd schreibtisch
git clone https://github.com/pradhanshrijal/pha_docker_files
```

Now we can create a docker image.

```bash
cd /home/${USER}/schreibtisch/pha_docker_files
docker build -t phaenvs/pha-22:mini-sample -f envs/pha-22-mini/Dockerfile --no-cache .
```

So we build our docker image by specifying the name with `-t, --tag` where `name:tag` is the convention. We specify the file with the `-f, --file` option. `--no-cache` option is used to remove the cache to reduce the size. When you are using a path to build an image then you use the `.`. 

---
## Docker Hub

Docker Hub is a cloud-based storage and sharing platform specifically designed for container images (*Powered by [Gemini][Gemini]*).

Some samples of the images can be found in [phaenvs]. The definitions of the images are available in the [PHA Git Wiki - Images].

---
## User from the Host Machine

What we have learned till now generally works very well. But there are cases when docker permissions are associated to a particular user. This means in an Ubuntu system with several users, the current host is not the one to whon docker permissions are associated with. In such cases, to use the [SSI] we have to perform one additional step. This is to pass the user from the host machine to docker. 

In this case, we have to create a specialized docker image by passing the user specific information to the image.

This method is available as a seperate `env` in [PHA Git], called `custom-user`. 

#### Set the variables

```dockerfile
# Declare VARIABLES
ARG IMAGE_NAME=phaenvs/pha-22
ARG IMAGE_VERSION=latest
ARG DEBIAN_FRONTEND=noninteractive

FROM ${IMAGE_NAME}:${IMAGE_VERSION} as base
FROM base as base-amd64
```

We set the name and tag of the image we want to use as variable and we make sure we do not have to interract with any installations. The we select the `amd64` version of the image as that is what we will be working with. 

#### User Details

```dockerfile
USER root

ARG USERNAME=devuser
ARG UID=${UID}
ARG GID=${GID}

# Remove User
RUN userdel pha
```

What we did here is first to make sure that the current user is `root` so as to make sure we are creating the new user properly. Then we set the variables for the username, *UID* and *GID* of the user we want to pass to docker. We also have to delete any current user incase there is an id conflict.

#### Create User

```dockerfile
# Create new user and home directory
RUN groupadd --gid $GID $USERNAME \
 && useradd --uid ${UID} --gid ${GID} --create-home ${USERNAME} \
 && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} \
 && chmod 0440 /etc/sudoers.d/${USERNAME} \
 && mkdir -p /home/${USERNAME} \
 && chown -R ${UID}:${GID} /home/${USERNAME}

USER ${USERNAME}
WORKDIR "/home/${USERNAME}"
```

We then create the new docker user with the user variable that were set in the previous section. This makes sure that our docker user and the host user have the same permission IDs. Finally, in our new image we set the new created user as the default and change the work directory to the home of this user. All this is documented well in [Set User Container Host].

---
## Conclusion

This article explains how a CUDA enabled image is created with the principles of the [PHA Project]. In the next article we will discuss some of the positives and negatives of this method and also provide a simpler application with `docker-compose` [[Chaotic Docker - Part III]].

---
## Bibliography

- [Docker ROS Guide]
- [Docker ROS 2 Guide]
- [PHA 22 Mini]
- [PHA Git]
- [PHA Git Wiki]
- [PHA Git Wiki - Images]
- [phaenvs]
- [Gemini]
- [CuDNN]
- [CUDA Gitlab]
- [ROS Humble]
- [Colcon Quick Directory]
- [Set User Container Host]

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[PHA Project]: {{site.url}}/pha-project/
[Chaotic Docker - Part I]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/
[Chaotic Docker - Part I - Hardware Drivers]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#hardware-drivers
[Chaotic Docker - Part I - SSI Structure]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#structure-of-the-ssi
[Chaotic Docker - Part I - Finally, Space for the SSI]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#finally-space-for-the-ssi
[Chaotic Docker - Part III]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-iii/
[PHA 22 Mini]: https://hub.docker.com/layers/phaenvs/pha-22/mini/images/sha256-9a6281b350f1d279374f28fc5d9b70e0996b1f6b588593ae37b622c09d58ca74?context=explore
[PHA Git]: https://github.com/pradhanshrijal/pha_docker_files
[PHA Git Wiki]: https://github.com/pradhanshrijal/pha_docker_files/wiki
[PHA Git Wiki - Images]: https://github.com/pradhanshrijal/pha_docker_files/wiki/PHA-Images
[phaenvs]: https://hub.docker.com/u/phaenvs
[Gemini]: https://gemini.google.com/
[CuDNN]: https://developer.nvidia.com/cudnn
[CUDA Gitlab]: https://gitlab.com/nvidia/container-images/cuda
[ROS Humble]: https://docs.ros.org/en/humble/index.html
[Colcon Quick Directory]: https://colcon.readthedocs.io/en/released/user/installation.html#quick-directory-changes
[Set User Container Host]: https://www.baeldung.com/ops/docker-set-user-container-host
---