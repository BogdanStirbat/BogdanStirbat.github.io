---
layout: post
title:  "Docker Architecture"
date:   2023-04-29 12:16:18 +0300
categories: jekyll update
---

[Docker](https://www.docker.com/) is a platform for developing, shipping and running containerized applications. Here is an overview of the Docker architecture:

![Project structure]({{ "/assets/2023-04-29-docker-architecture/docker-architecture-overview.png" | absolute_url }})

As we can see, the architecture consists of a Client, a Docker Host, a Registry. 

Docker uses a client-server architecture. The Docker Client is used for sending commands to the Docker Host; on the Docker Host runs a Docker daemon, responsible for building, running, distributing Docker objects.  

The Docker daemon is named dockerd. It listens for Docker API requests (from the Docker client) and manages Docker objects. 
Example Docker objects:
 - Images
 - Containers
 - Networks
 - Volumes

The Docker Registry is the place where Docker images are stored. The Docker registry is a repository for Docker images. By default, the Docker registry is [Docker Hub](https://hub.docker.com/search?q=). Here can be found different images, built by different entities. Of course, if needed, Docker can be configured to use a private registry.

A Docker image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some customization.
The first step in creating a Docker image is creating a Dockerfile. A Dockerfile is a file containing syntax for defining the steps needed for creating and running an image. Each instruction in a Dockerfile creates a layer in the image.
When the Dockerfile is changed and the image is rebuild, only the changed layers are rebuilt. 

A container is a runnable instance of an image. A container can be connected to one or more Docker networks, storage can be attached to it, or a new image can be created from a container.
When a new container is created, the Docker image is retrieved from the local registry. If the image doesn't exist on the local registry, the image will be pulled from the Docker Registry (either DockerHub, either a private registry). 

Networks allow containers to communicate with each other, or the host machine to communicate with a container.

All data on a Docker container is tied to the lifecycle of that specific container; once a container is deleted, all container data is deleted as well.
To prevent this, Docker Volumes can be created and attached to containers. Once a container is deleted, the Docker volume still persists its data.


### Links

https://learn.acloud.guru/course/06875cfa-fb88-4b28-ac90-150e73bb5a85/

https://docs.docker.com/get-started/overview/

https://www.aquasec.com/cloud-native-academy/docker-container/docker-architecture/

https://sysdig.com/learn-cloud-native/container-security/what-is-docker-architecture/
