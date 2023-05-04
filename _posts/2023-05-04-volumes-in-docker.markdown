---
layout: post
title:  "Volumes in Docker"
date:   2023-05-04 11:16:18 +0300
categories: jekyll update
---

As we previously saw, when a new Docker container is started from a Docker image, a new R/W layer is appended to the Docker container.
Any changes made to the container will be persisted in the R/W container; once the container is removed, the R/W container is removed as well.
This can be a problem, since data is not persistent across containers.

To solve this problem, Docker comes with new types of objects called volumes. A volume can be attached to a Docker container, and the lifecycle of a Docker volume is independent of the lifecycle of the container.

To create a new volume:
```
$ docker volume create test_volume
test_volume
```

To list existing volumes:
```
$ docker volume ls
DRIVER    VOLUME NAME
local     test_volume
```

Now, let's create a new container, attaching the volume `test_volumme` to it: 
`docker container run -it --name test_container --mount type=volume,source=test_volume,target=/var/lib busybox`. 
This command starts a new container, mounting the volume `test_volume` in the container's filesystem, in the path `/var/lib`.

### Bind mounts

Bind mounts are a functionality that allow a volume to be mounted both in a Docker container and on the host filesystem. 

To make a bind mount, let's first make a new folder, named `test_bindmounts_folder`. Go to that folder, and in that folder execute:
```
$ docker container run -d --name test-bind-mount --mount type=bind,source="$(pwd)"/test_bindmounts_folder,target=/app nginx
```
This command mounts the folder `test_bindmounts_folder` into the container's filesystem, in the location `/app`. To test this, execute in the `test_bindmounts_folder` folder:
```
$ echo "Hello from host" > target/file.txt
```

Now, let's connect to the created container, to inspect the filesystem: 
```
$ docker container exec -it test-bind-moun
```

```
# cat app/file.txt
Hello from host
```

From this example, we saw that a bind mount is a volume that is mounted both on the host file system and on the container's file system.

### Conclusions
Data produced in a Docker container is lost once the container is deleted. To solve this problem, volumes are new Docker objects that allow data tp be persistent across Docker containers. 

Using bind volumes, data can be mounted both on the host filesystem and on the container's filesystem. This can be very useful, for example, for debugging purposes.

### Links

https://learn.acloud.guru/course/06875cfa-fb88-4b28-ac90-150e73bb5a85/

https://docs.docker.com/storage/volumes/

https://earthly.dev/blog/docker-volumes/#top

https://www.baeldung.com/ops/docker-volumes
