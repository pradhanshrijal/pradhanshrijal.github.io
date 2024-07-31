---
layout:     post
title:      Surfing Carlaware
subtitle:   Guide to Simplifying Carla and Autoware
date:       2024-07-31
author:     Shrijal Pradhan
ext-img: "https://drscdn.500px.org/photo/1097881280/q%3D90_m%3D2048/v2?sig=16a79c4119130f488c95a6ea4409cab6582e82ce2b9fb56eccd7a369419b25aa"
catalog: true
categories: blog
permalink: /:categories/:title/
tags:
    - pha
    - carla
    - docker
    - robotics
    - ros
---

## Introduction

We will build upon the easy intructions provided by [PHA Carlaware - L1] on how to work with [PHA Carlaware]. This is the L2 set of instructions that are made to be simple but requires the user to understand some simple parameters that can be modified to make changes to the project. So in a manner the user can smoothly surf through the project by understanding the basics of how the project is organised. 

The remainder of in the article is organized in the following manner:
- Introduction to PHA
- Folder Structure for SSI
- Carlaware as a PHA Addon
- Modifiable Variables
- Changing simple install scripts
- Expanding CWR with ROS Repos 

---
## Introduction to PHA

[PHA Project] just uses scripts to run commands. By switching which scripts are run, a very flexible and powerful system can be build with a very small number of changes. PHA can be understood as a bunch of cords that is used to bind together several modular systems and make it easy to change any of these modules. PHA is setup at `/home/${USER}/schreibtisch/pha_docker_files` which we will call `${PHA_HOME}`.

---
## Folder Structure for SSI

To make these modules easily accessible, PHA defines several environment variable. In addition, PHA assigns a single folder that is accessible all docker containers created in the network. This helps the containers to share critical parameter information and large file systems within each other.

This path is `${PHA_HOME}/docker_share` by default for Carlaware, we will call this `${SSI_PATH}` for the remaining article.

See [SSI] for more details.

---
## Carlaware as a PHA Addon

PHA provides a space for Addons, these addons are used to provide complex functions to PHA that are not part of the simplicity of PHA. In this case, the addon is Carlaware. Carlaware is defined in `${SSI_PATH}/git_pkgs/pha_addons/pha_carlaware`. We will call this space `${CARLAWARE_PHA}`.

---
## Modifiable Variables

With the above information, we will look deeper into the environment variables [ENV CWR] that we run when using [PHA Carlaware - L1]. The most important for our case are:

```
PROJECT_NAME="pha-carlaware"
CONT_NAME="${PROJECT_NAME}-${USER}"

APT_GET_REQUIREMENTS_FILE="apt-get_requirements.txt"
PYTHON_REQUIREMENTS_FILE="python_requirements.txt"
SCRIPT_REQUIREMENTS_FILE="script_requirements.sh"
USER_REQUIREMENTS_PATH="docker_share/git_pkgs/pha_addons/pha_carlaware/requirements"
```

The `CONT_NAME` is the name of the docker container. The user can change this to fit the function of the particular project or module (i.e. `pha-perception`).

The three `*_FILE` variables are the three files that define the automated installations. In `APT_GET_*` and `PYTHON_*`, it is as simple as adding the name of the package to be installed by the respective methods in a new line. The `SCRIPT_*` space is a bit unique. In this file the user can write commands in a new line as one would in the bash terminal.

> **_NOTE:_**
> Make sure there are no conflicts or user inputs required for the commands (i.e. use that tag `-g` when installing something with apt).

The `USER_REQUIREMENTS_PATH` is the variable that points to where these installation files are. Keep in mind tha the file is not the absolute path but the path pointed from `${PHA_HOME}`.

The user can change these files, including [ENV CWR] if the requirements of the application changes.

## Running the System

Based on the information provided, now we can look at the run scripts again.

#### Setup Carlaware

We setup the same way as [PHA Carlaware - L1]:

```
wget https://raw.githubusercontent.com/pradhanshrijal/pha_carlaware/master/scripts/setup_cwr.sh
source setup_cwr.sh
```

> **_NOTE:_**
> Run Carla and Carlaware on separate computers for the best performance.

#### Start Carla

A simple run script is provided to run `Carla 0.9.15`.

```
cd $CARLAWARE_PHA/scripts
./carla_run.sh
```

#### Setup Carlaware

The whole system can be setup with a single script. For the first run it take a while to complete the setup process.

```
cd $CARLAWARE_PHA/scripts
source run_cwr.sh
```

For systems with multiple GPUs like the [NVIDIA DGX Platform] there is a sample script that run only the first two GPUs `0,1`.

```
cd $CARLAWARE_PHA/scripts
source run_cwr_gpus.sh
```

All these scripts do is run a command that references two files: an environment file (that follows `-e`) and a file to connect the environmental variables to docker (that follows `-b`). If the user needs to change something, it is recommended to make a copy of [PHA CARLAWARE] and change the paths to the respective file. The variables should also be changed in the environment file to point to the correct path as explained in [Modifiable Variables](#modifiable-variables). The actual command is as follows:

```
${PHA_HOME}/docker_scripts/run-compose.sh \
    -b ${CARLAWARE_PHA}/envs/cwr-compose.yaml \
    -e ${CARLAWARE_PHA}/envs/op-user.env
```

> **_NOTE:_**
> Once the setup is complete, running the same command will reinitialize the setup.
> To restart the previous setup (i.e. when the pc is started again), run this command from the second time:
> `docker start pha-carlaware-${USER}`

Now you can enter the setup container:

```
docker exec -it pha-carlaware-${USER}
```

This is a space where all the setup in ready for the user.

#### Launch ROS

To Launch the script, you have to go to the respective folder and run the launch script.

```
cd $OP_BRIDGE_ROOT/op_scripts
```

> **_NOTE:_**
> If you are running the system of Two PCs, change IP of `SIMULATOR_LOCAL_HOST` to that of the Carla PC. 
> Make sure you can ping the other PC.

Launch Carlaware.

```
./cwr_exploration.sh
```

---
## Bibliography

- [ENV CWR]
- [PHA Carlaware]
- [PHA Docker]

[Carla 0.9.15 Consumption]: {{site.url}}/{{page.categories}}/carla-0.9.15-consumption/
[ENV CWR]: https://github.com/pradhanshrijal/pha_carlaware/blob/master/envs/op-user-gpus.env
[PHA Project]: {{site.url}}/pha-project/
[PHA Carlaware]: https://github.com/pradhanshrijal/pha_carlaware
[PHA Carlaware - L1]: https://github.com/pradhanshrijal/pha_carlaware#installation-l1
[SSI]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#single-source-of-information
[PHA Docker]: https://github.com/pradhanshrijal/pha_docker_files 