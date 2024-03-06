---
layout:     post
title:      A Chaotic Guide to Using Docker for Robotics Research - Part IV
subtitle:   TL;DR, Just Use Docker Compose
date:       2024-03-04
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

## Introduction to Compose
- `/dev` files can be individually assigned an ID and sent to docker
- No comment on RVIZ

## White Paper Criticism
- No proper alternative given
- Kinda rejected by the community
- Why not create non-sudo (and sudo) users in docker with their own passwords?
- The white paper does not provide proper counter arguments
- Not enough references or extensive examples of what they consider breaches

## Advantages
- Different research institues have different installation methods
- Different projects have different demands

## Disadvantages
- Opened Access

## Conclusion

This article creates a base platform for the [PHA Project] where different modules of the automated driving stack could be build upon.

## References

- [Docker ROS Guide]
- [Docker ROS 2 Guide]
- [Docker run]
- [Ubuntu White Paper]
- [PHA 22 Mini]
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
- [CuDNN]
- [Realsense SDK Guide]
- [Shared Memory]

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Docker run]: https://docs.docker.com/reference/cli/docker/container/run/
[Docker network]: https://docs.docker.com/reference/cli/docker/network/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/
[PHA 22 Mini]: https://hub.docker.com/r/phaenvs/pha-22-mini
[PHA Docker]: https://github.com/pradhanshrijal/pha_docker_files 
[Easy Guide to Installing Docker]: {{site.url}}/blog/easy-guide-to-installing-docker/
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/
[Container GUI]: https://leimao.github.io/blog/Docker-Container-GUI-Display/
[Docker GUI]: https://linuxmeerkat.wordpress.com/2014/10/17/running-a-gui-application-in-a-docker-container/
[X11 Forwarding]: https://forums.docker.com/t/x11-forwarding-with-v-on-docker-run-not-working/17708
[ROS Docker GUI]: https://wiki.ros.org/docker/Tutorials/GUI
[Specialized Docker]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
[Container Runtime]: https://developer.nvidia.com/container-runtime
[Gemini]: https://gemini.google.com/
[Why GPUs]: https://blogs.nvidia.com/blog/why-gpus-are-great-for-ai/
[CuDNN]: https://developer.nvidia.com/cudnn
[CUDA Gitlab]: https://gitlab.com/nvidia/container-images/cuda
[Realsense SDK Guide]: https://dev.intelrealsense.com/docs/compiling-librealsense-for-linux-ubuntu-guide
[Shared Memory]: https://www.cyberciti.biz/tips/what-is-devshm-and-its-practical-usage.html