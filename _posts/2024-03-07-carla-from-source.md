---
layout:     post
title:      Carla from Source
subtitle:   PHA and Carla
date:       2024-03-07
author:     Shrijal Pradhan
header-img: img/post-maspalomas.jpg
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

In this article, we will install carla from source with the help of docker and the [PHA Project]. What we will do is install Carla and Unreal Engine in the [SSI]. In the last article [Carla 0.9.15 Consumption] we looked at how the [PHA Project] can be used for flexible installation of any ROS based package. In this article we will look at how other softwares can be installed with docker. This example will be using `make` for installations.

---
## Patching Carla for Ubuntu 22.04

To make carla work with Ubuntu 22.04, thre are two changes that are necessary. First, the LLVM version would have to be upgraded to `LLVM-12.0` as documented in [Carla Under Ubuntu 22.04]. Further as Ubuntu 22.04 does not include python2, the python version must also be upgraded to `python3`.

#### LLVM-12.0 Update

Change `carla/Util/BuildTools/Setup.sh:73` to:

```bash
LLVM_BASENAME=llvm-12.0
```

if this does not exist then change to line 54:

```bash
CXX_TAG=c12
```
 
#### Python3 Update

This project uses Python 3.10.

Change `carla/Util/BuildTools/BuildCarlaUE4.sh:134` and `carla/Util/BuildTools/BuildCarlaUE4.sh:137` to

```bash
python3
```

Alternatively they may be in `carla/Util/BuildTools/BuildCarlaUE4.sh:163` and `carla/Util/BuildTools/BuildCarlaUE4.sh:166`.

If you want to spawn walkers with python 3.9, you should change `carla/PythonAPI/carla/source/libcarla/World.cpp` Line 305 (or Line 314 in [8854804]) from

```bash
.def("get_random_location_from_navigation", CALL_RETURNING_OPTIONAL_WITHOUT_GIL(cc::World, GetRandomLocationFromNavigation))`
```

to

```bash
.def("get_random_location_from_navigation", CALL_RETURNING_OPTIONAL(cc::World, GetRandomLocationFromNavigation))
```

This is documented in [Generate Traffic 3.10].

Then, delete in `carla/PythonAPI/carla/source/libcarla/Client.cpp` Line 25 (or 28 in [8854804])

```bash
carla::PythonUtil::ReleaseGIL unlock;
```

will solve this problem.

Further, change `carla/PythonAPI/examples/requirements.txt` and `carla/PythonAPI/carla/requirements.txt` from

```bash
numpy; python_version < '3.0'
numpy==1.18.4; python_version >= '3.0'
```

to

```bash
numpy; python_version >= '3.0'
```

#### Available Patch

The patch is available in [Carla Ubuntu 22 Patch].

---
## Pre-Installations

We will start by installing the dependencies required for Carla.

#### Unity Permissions

Before starting with the installations, be aware that to download the Carla fork of Unreal Engine, you need to have a GitHub account linked to Unreal Engine's account. If you don't have this set up, please follow this guide [UE on Github] before going any further. The commit is `8d08ab7fe891737b7badebdc77333c24b8348ca1`

#### Install Dependencies

Dependencies for Unreal Engine and Carla are available in a neatly packed script [install_carla_deps.sh]. The content of the scripts are as follows:

```bash
#!/bin/bash

# source install_carla_unreal.sh /home/${USER}/docker_share/git_pkgs/simulators

# Variables
IN_CARLA_UNREAL_FOLDER=$1
IN_CARLA_VERSION=$2
IN_CARLA_UNREAL_FOLDER="${IN_CARLA_UNREAL_FOLDER:=/home/${USER}/docker_share/git_pkgs/simulators}"
IN_CARLA_VERSION="${IN_CARLA_VERSION:=0.9.15}"
IN_CARLA_ROOT=${IN_CARLA_UNREAL_FOLDER}/carla
IN_UE4_ROOT=${IN_CARLA_UNREAL_FOLDER}/UnrealEngine_4.26
source /home/${USER}/.bashrc
cd ${IN_CARLA_UNREAL_FOLDER}

# Downloads
git clone --depth 1 -b carla https://github.com/CarlaUnreal/UnrealEngine.git UnrealEngine_4.26
git clone https://github.com/pradhanshrijal/carla -b feature/u22-${IN_CARLA_VERSION}

# Install Unity
cd UnrealEngine_4.26
./Setup.sh && ./GenerateProjectFiles.sh && make

echo -e "\n# Unreal | Carla"
echo "export UE4_ROOT=${IN_UE4_ROOT}" >> /home/${USER}/.bashrc
echo "export CARLA_ROOT=${IN_CARLA_ROOT}" >> /home/${USER}/.bashrc
echo "export PYTHONPATH=\$PYTHONPATH:${IN_CARLA_ROOT}/PythonAPI/carla/dist/carla-${IN_CARLA_VERSION}-py3.10-linux-x86_64.egg:${IN_CARLA_ROOT}/PythonAPI/carla" >> /home/${USER}/.bashrc
source /home/${USER}/.bashrc

# Install Carla
cd ..
cd carla
./Update.sh
```

These changes are all available in a neatly packed docker image [PHA 22 Carla Mini].

---
## Primary Installations

We can now connect everything together. For setting up the [SSI] we will use [PHA Docker].

#### Folder Setup

First we setup the project as documented in [SSI Space].

```bash
cd /home/${USER}
mkdir schreibtisch
cd schreibtisch
git clone https://github.com/pradhanshrijal/pha_docker_files
cd pha_docker_files
```

#### Start the Container

```bash
./run.sh -c pha-22-carla-sample -i phaenvs/pha-22 \
      -s /home/${USER}/schreibtisch/pha_docker_files/docker_share \
      -t carla-mini 
```

#### Enter the Container

```bash
./docker_scripts/exec-cont.sh -c pha-22-carla-sample 
```

#### Install Unreal Engine and Carla

The installations are neatly packed into the script [install_carla_unreal.sh].

```bash
source docker_share/scripts/install_carla_unreal.sh
```

> **_WARNING:_**
> If the UID an GID of the docker user and host user does not match [1000] then it will be problems.

The content of the scripts are as follows:

```bash
#!/bin/bash

# source install_carla_unreal.sh /home/${USER}/docker_share/git_pkgs/simulators

# Variables
IN_CARLA_UNREAL_FOLDER=$1
IN_CARLA_VERSION=$2
IN_CARLA_UNREAL_FOLDER="${IN_CARLA_UNREAL_FOLDER:=/home/${USER}/docker_share/git_pkgs/simulators}"
IN_CARLA_VERSION="${IN_CARLA_VERSION:=0.9.15}"
IN_CARLA_ROOT=${IN_CARLA_UNREAL_FOLDER}/carla
IN_UE4_ROOT=${IN_CARLA_UNREAL_FOLDER}/UnrealEngine_4.26
source /home/${USER}/.bashrc
cd ${IN_CARLA_UNREAL_FOLDER}

# Downloads
git clone --depth 1 -b carla https://github.com/CarlaUnreal/UnrealEngine.git UnrealEngine_4.26
git clone https://github.com/pradhanshrijal/carla -b feature/u22-${IN_CARLA_VERSION}

# Install Unity
cd UnrealEngine_4.26
./Setup.sh && ./GenerateProjectFiles.sh && make

echo -e "\n# Unreal | Carla"
echo "export UE4_ROOT=${IN_UE4_ROOT}" >> /home/${USER}/.bashrc
echo "export CARLA_ROOT=${IN_CARLA_ROOT}" >> /home/${USER}/.bashrc
echo "export PYTHONPATH=\$PYTHONPATH:${IN_CARLA_ROOT}/PythonAPI/carla/dist/carla-${IN_CARLA_VERSION}-py3.10-linux-x86_64.egg:${IN_CARLA_ROOT}/PythonAPI/carla" >> /home/${USER}/.bashrc
source /home/${USER}/.bashrc

# Install Carla
cd ..
cd carla
./Update.sh
```

Keep in mind that this script also automates the root path export of Unreal Engine and Carla via the `.bashrc`.

---
## Build and Launch Carla

Navigate to the carla folder:

```bash
cd $CARLA_ROOT
```

Build and launch carla, this has to be done everytime the system is started.

```bash
make PythonAPI
make launch -opengl
```

---
## Known Issues

#### Using Vulkan to launch Carla

The normal way to launch Carla is with vulkan. With this installation method, the system crashes when running this way. This is the reason the launch method uses the `-opengl` flag. See [Can't Launch Carla].

#### Visualization of Client running at very low FPS Rate

Set the UE Editor to run on high rate, even when idle. Go to `Edit -> Editor preferences -> Performance` in the Unreal Engine editor and disable `Use less CPU when in background`. See [Carla FPS Rate].

#### Changing Map

When changing map there are several issues with Carla. The CPU load explodes, no reason for this has been analyzed. However, there is a possible solution. As PHA is a very scripts based project, the solution is in the form of two scripts; [carla_map_update.sh] and [pha_carla_vars.sh].

[pha_carla_vars.sh] registers the current map and [carla_map_udpate.sh] updates the map. Very simple, this assigns the default map so the scripts have to be run before the carla system is started. Make sure `$CARLA_ROOT/Unreal/CarlaUE4/Config/DefaultEngine.ini` and [pha_carla_vars.sh] have the same starting value for the map.

```bash
source /home/${USER}/docker_share/scripts/functions/carla_map_update.sh 01
```

---
## Bibliography

- [Generate Traffic 3.10]
- [Carla Ubuntu 22 Patch]
- [UE on Github]
- [install_carla_deps.sh]
- [install_carla_unreal.sh]
- [carla_map_update.sh]
- [pha_carla_vars.sh]
- [PHA 22 Carla Mini]
- [PHA Docker]
- [Can't Launch Carla]
- [Carla FPS Rate]
- [8854804]

[Carla 0.9.15 Consumption]: {{site.url}}/{{page.categories}}/carla-0.9.15-consumption/
[PHA Project]: {{site.url}}/pha-project/
[SSI]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#single-source-of-information
[Carla Under Ubuntu 22.04]: https://github.com/carla-simulator/carla/issues/6381#issuecomment-1560837119
[Generate Traffic 3.10]: https://github.com/carla-simulator/carla/issues/5658#issuecomment-1213121647
[Carla Ubuntu 22 Patch]: https://github.com/pradhanshrijal/carla/tree/feature/u22-0.9.15
[UE on Github]: https://www.unrealengine.com/en-US/ue-on-github
[install_carla_deps.sh]: https://github.com/pradhanshrijal/pha_docker_files/blob/master/docker_share/scripts/install/install_carla_deps.sh
[install_carla_unreal.sh]: https://github.com/pradhanshrijal/pha_docker_files/blob/master/docker_share/scripts/install/install_carla_unreal.sh
[carla_map_update.sh]: https://github.com/pradhanshrijal/pha_docker_files/blob/master/docker_share/scripts/functions/carla_map_update.sh
[pha_carla_vars.sh]: https://github.com/pradhanshrijal/pha_docker_files/blob/master/docker_share/scripts/vars/pha_carla_vars.sh
[PHA 22 Carla Mini]: https://hub.docker.com/layers/phaenvs/pha-22/carla-mini/images/sha256-cbf40301313a0f808e656ccac08122c2079d7edd501d8cd68380eed99b5e8149?context=explore
[PHA Docker]: https://github.com/pradhanshrijal/pha_docker_files 
[Can't Launch Carla]: https://github.com/carla-simulator/carla/issues/2138#issuecomment-616114548
[Carla FPS Rate]: https://github.com/carla-simulator/carla/issues/3654#issuecomment-737042264
[8854804]: https://github.com/carla-simulator/carla/commit/8854804f4d7748e14d937ec763a2912823a7e5f5