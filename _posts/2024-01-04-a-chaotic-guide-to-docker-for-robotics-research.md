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

This article delves into a new approach to using Docker. An approach that will supercharge Docker for a large robotics stack. We have to first ask if there is the need for additional features when using Docker for robotics research, specifically with ROS. We also have to conform a stable driver base within th main operating system for the usage of the docker containers.

This article creates a base platform for the [PHA Project][PHA Project] where different modules of the automated driving stack could be build upon.

[Docker ROS Guide]: https://roboticseabass.com/2021/04/21/docker-and-ros/
[Docker ROS 2 Guide]: https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
[Ubuntu White Paper]: https://ubuntu.com/engage/dockerandros
[PHA Project]: {{site.url}}/pha-project/