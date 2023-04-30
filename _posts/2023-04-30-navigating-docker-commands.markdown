---
layout: post
title:  "Navigating the Docker commands"
date:   2023-04-30 12:16:18 +0300
categories: jekyll update
---

[Docker](https://www.docker.com/) is a platform for developing, shipping and running containerized applications. To get an overview of the available commands, type `docker --help`. 
You will see a list of available commands, grouped into categories. For example, we have common commands, management commands.

The management commands manage the Docker objects. For example, to check how we can manipulate Docker images, we can type `docker image --help`, to check how we can manipulate containers, we can type `docker container --help`.

From the output of the `--help` commands, we learn that, to list images, we can type `docker image ls`, and to list containers we can type `docker container ls`. Further options can be observed by executing these commands adding the `--help` argument. 

By navigating the Docker commands like this, one parson can learn how to manage Docker objects.

### Links

https://learn.acloud.guru/course/06875cfa-fb88-4b28-ac90-150e73bb5a85/

https://docs.docker.com/engine/reference/commandline/docker/

https://buddy.works/tutorials/docker-commands-cheat-sheet
