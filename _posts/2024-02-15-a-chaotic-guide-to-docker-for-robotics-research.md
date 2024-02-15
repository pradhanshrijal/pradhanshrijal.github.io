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

P.S. Install Docker with [Easy Guide to Installing Docker].

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

- [Docker ROS Guide]
- [Docker ROS 2 Guide]
- [Ubuntu White Paper]
- [Using GPU in Docker]

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/
[Easy Guide to Installing Docker]: {{site.url}}/blog/easy-guide-to-installing-docker/
[Using GPU in Docker]: https://blog.roboflow.com/use-the-gpu-in-docker/