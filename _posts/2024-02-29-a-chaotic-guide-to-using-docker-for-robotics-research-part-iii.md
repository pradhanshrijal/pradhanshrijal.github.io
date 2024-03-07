---
layout:     post
title:      A Chaotic Guide to Using Docker for Robotics Research - Part III
subtitle:   Comments and Docker Compose
date:       2024-02-29
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

This article is the conclusion to the Chaotic Docker Series, which creates a basis for the PHA Project. Further, it provides an easy way for applying the PHA way for people who do not want to spend a few days to understand Docker and want to directly use it.

## Advantages

The goal of Single Source of Information ([SSI]) is to provide a platform where different software packages can be used modularly in their dockerized forms for different systems. This means that it is a very flexible development platform that would be robust to changes that come during any development process. Another advantage of the SSI is that since this space is shared between the host and the container, no credentials have to be passed to the container. Any changes to the softwares can be made from the host system.

## Disadvantages

The modularity based on several parameter definitions means the system would be very modular but it also increases the complexity of the project. Most of the modules would be dependant on several parameters that communicate very small details of the system. The biggest question about this article is that it opens access to several hardware resources of the host system that can make the system vurnerable. For this reason, the project is as of this point only ready for development, it is not production ready. However, with some security based decisions regading the access of the hardware resources, the system can be more secure. Further, if any source installations are made on the SSI, then this does not associate to the size of the container.

## Alternative

One alternative to the docker ros robotics stack concept is [IKA ROS] and [IKA ROS ML]. They provide several different versions of ROS Docker and also provide several functions for installations. However, the author would argue (and of course the author tries to sell their idea), that with the SSI these installations could be easily applied to the container. This means that there are no issues with permissions when pulling or pushing with git, but again this project gives a lot of permissions to the containers. Further, if we install with one container, we can also use the software for a container created with the same image. But PHA would not be a consuptions project without look at alternatives without thinking about integrating them, this could be an idea for the future.

## TL;DR Docker Compose

This section would be short and sweet. If anyone wants more information, see [Docker Compose]. 

#### Introduction

Just remember:

| **Command**| **Definition** |
| `up` | Initialize and start the docker container |
| `down` | Stop and remove the docker container |
| `start` | Onyl start an available docker container  |
| `stop` | Only stop a docker container |

#### SSI, Folder Structure

If you have not created the folder structure as explained in [Space for the SSI], setup the folder structure before we create the container:

```bash
xhost +local:docker
cd /home/${USER}
mkdir schreibtisch
cd schreibtisch
git clone https://github.com/pradhanshrijal/pha_docker_files
```

#### Sample Usage - Compose

Sample up:

```bash
docker-compose -f envs/pha-22-mini/docker-compose.yaml up -d
```

Sample down:

```bash
docker-compose -f envs/pha-22-mini/docker-compose.yaml down
```

For `start` and `stop` it's the same as down.

#### Sample Usage - Container

With this method we are running docker in the background, now we will look at some additional functions:

Entering Docker:

```bash
docker exec -it pha-22-mini /bin/bash
```

Stopping Docker:

Don't forget to exit the container if you are inside of it.

```bash
docker stop pha-22-mini
```

Start Docker:

Once you have initialized the container with `docker run` you don't have to run the command over and over again, We are storing it for future use:

```bash
docker start pha-22-mini
```

#### Workspace for Users

Users can install their packages make changes in `/home/${USER}/schreibtisch/pha_docker_files/docker_share/git_pkgs`. This folder will not disappear if you wish to use docker like an acid bath technique and clear it out for a new fresh installation. It is hight recommended to use a version control system like git to maintain and save your work.

## Conclusion

This article series creates a base platform for the [PHA Project] where different modules of the automated driving stack could be built upon. It gives an extensive guide to using docker and ends with a super short usage guide for docker.

## References

- [PHA 22 Mini]
- [PHA Docker]
- [Gemini]
- [IKA ROS ML]
- [IKA ROS]
- [Docker Compose]

[PHA Project]: {{site.url}}/pha-project/
[PHA 22 Mini]: https://hub.docker.com/r/phaenvs/pha-22-mini
[PHA Docker]: https://github.com/pradhanshrijal/pha_docker_files 
[Easy Guide to Installing Docker]: {{site.url}}/blog/easy-guide-to-installing-docker/
[SSI]: {{site.url}}/blog/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#single-source-of-information
[Space for the SSI]: {{site.url}}/{{page.categories}}/a-chaotic-guide-to-using-docker-for-robotics-research-part-i/#finally-space-for-the-ssi
[Gemini]: https://gemini.google.com/
[IKA ROS ML]: https://github.com/ika-rwth-aachen/docker-ros-ml-images
[IKA ROS]: https://github.com/ika-rwth-aachen/docker-ros
[Docker Compose]: https://docs.docker.com/compose/