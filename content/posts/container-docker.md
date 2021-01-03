---
title: "What is container and what is docker?"
date: "2020-11-15"
description: "Docker and container overview"
tags: [container, docker, devops]
categories: [devops, container]
---

This article some introductory lession on docker and container.
<!--more-->

### What is docker?

It is a powerful tool which allows devops (system administrator, developers) to deploy application from one environment to another environment without considering the host operating system.

The main benefit of docker is that it packages application in a sandboxed environment with all of its packages and dependencies as a standard unit.

If we compare between docker container and virtual machines, containers have less overheads that traditional vms and can make more proper use of system and resources.

### What is containers?

It is just a process running in the operating system with its own configurations. You can see the container process ID using

ps -aux | grep “name_of_container”

Then go to /proc/PROCESS_ID

Instead of using virtual machines, we use containers. VM run on application inside a guest operating system, which runs on virtual hardware powered by the server host OS.

### Why use containers?

Container offer a logical packaging mechanism in which an applications can be abstracted from the environment in which they actually run.

#### Running hello-world container

Docker run hello-world

This command check whether you have hello-world image in your local repository and if not found it pulls the image from docker hub/registry and then starts the container

#### Pulling docker from registry

Docker pull busybox

This command fetches the busy box image from docker registry and saves it to our system.

Docker run

Now in order to run a docker container from your image, we just issue a command

Docker run busybox

See all the running container

In order to see all the running container we can see the followign command

Docker ps

If you want to see that list of containers that we ran before then we use the command

Docker ps -a

Accessing docker container

In order to access the container you need to issue following command

Docker run busybox -it

Deleting docker container

Docker rm (hash for the docker container obtained from docker ps -a)

This information should get you going on docker and provide you and overview of what docker is and what small tasks that we can do it. Later we can see some advanced things that we can do in docker.

Stay tuned and keep learning