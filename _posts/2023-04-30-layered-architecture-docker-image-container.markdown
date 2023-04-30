---
layout: post
title:  "The layered architecture of Docker images and containers"
date:   2023-04-30 14:16:18 +0300
categories: jekyll update
---

[Docker](https://www.docker.com/) is a platform for developing, shipping and running containerized applications.
An application runs inside a container, and each container is created from an image. If we use the OOP analogy, an image is a class, and a container is an instance of that class. 

## The layered architecture of Docker images

An image is build from a Dockerfile. In the Dockerfile there are different instructions. Each instruction in the Dockerfile adds a new layer.
A Docker image consists of different layers, each layer corresponding to a different Dockerfile instruction.

To show this in practice, let's create a file named `Dockerfile` with the content below:
```
FROM httpd:2.4
RUN apt update -y && apt upgrade -y && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists*
```

This `Dockerfile` defines a new image, based on the `httpd:2.4` image. To actually create this image, run in the same directory: `docker build -t dockerlayers:0.1 .`

This creates a ne image, `dockerlayers:0.1` . To prove that this image is based on the `httpd:2.4` image, let's first inspect the layers of the `httpd:2.4` image, then the layers of the `dockerlayers:0.1` image.

Inspect the layers of the `httpd:2.4` image:
```
$ docker inspect -f "{% raw %}{{ range .RootFS.Layers }}{{ println . }}{{end}}{% endraw %}" httpd:2.4
sha256:ed7b0ef3bf5bbec74379c3ae3d5339e666a314223e863c70644f7522a7527461
sha256:568467bc0db529f94b52d3cfff6110cfdaf996fac92e27e0a00d052a644f6e8b
sha256:16f81c3ee33ae40dac0f3ffa6771829eda3ea2b6f3ded534cc1b3b869a822272
sha256:e7bd853ca2e701ab964af00be463fc2a62f4c34a913deda78d90f4903c57627e
sha256:355053d0995ea7154219a87a700d654fcabf80d8586c52f8a19d5a036607413d
```

Inspect the layers of the `dockerlayers:0.1` image: 
```
$ docker inspect -f "{% raw %}{{ range .RootFS.Layers }}{{ println . }}{{end}}{% endraw %}" dockerlayers:0.1
sha256:ed7b0ef3bf5bbec74379c3ae3d5339e666a314223e863c70644f7522a7527461
sha256:568467bc0db529f94b52d3cfff6110cfdaf996fac92e27e0a00d052a644f6e8b
sha256:16f81c3ee33ae40dac0f3ffa6771829eda3ea2b6f3ded534cc1b3b869a822272
sha256:e7bd853ca2e701ab964af00be463fc2a62f4c34a913deda78d90f4903c57627e
sha256:355053d0995ea7154219a87a700d654fcabf80d8586c52f8a19d5a036607413d
sha256:0fab92a4da126eed2f6a50999be862d90baa8696a89ecee20fcd4dece5bd77c4
```

As we can see, the image `dockerlayers:0.1` consists of all the layers of the image `httpd:2.4`, plus an extra layer that ends with `5bd77c4`.

Now, let's edit the `Dockerfile`, and add a new instruction:
```
FROM httpd:2.4
RUN apt update -y && apt upgrade -y && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists*
RUN rm -f /usr/local/apache2/htdocs/index.html
```

Let's create the image `dockerlayers:0.2`: `docker build -t dockerlayers:0.2 .` . 
Let's also inspect the layers of the `dockerlayers:0.2` image:
```
$ docker inspect -f "{% raw %}{{ range .RootFS.Layers }}{{ println . }}{{end}}{% endraw %}" dockerlayers:0.2
sha256:ed7b0ef3bf5bbec74379c3ae3d5339e666a314223e863c70644f7522a7527461
sha256:568467bc0db529f94b52d3cfff6110cfdaf996fac92e27e0a00d052a644f6e8b
sha256:16f81c3ee33ae40dac0f3ffa6771829eda3ea2b6f3ded534cc1b3b869a822272
sha256:e7bd853ca2e701ab964af00be463fc2a62f4c34a913deda78d90f4903c57627e
sha256:355053d0995ea7154219a87a700d654fcabf80d8586c52f8a19d5a036607413d
sha256:0fab92a4da126eed2f6a50999be862d90baa8696a89ecee20fcd4dece5bd77c4
sha256:b9e6b9e3cb0a7a4a1e1796eba3bb3f6c3c5822caa395f356ddb6bb4417e042f7
```

We can see that the image `dockerlayers:0.2` consists of all the layers of the `dockerlayers:0.1` image, plus an extra layer.

Now, let's review the size of `dockerlayers:0.1`, and compare it with the size of `dockerlayers:0.2`. Since, in `dockerlayers:0.2`, the change is the deletion of a file, this should mean that `dockerlayers:0.2` is smaller than `dockerlayers:0.1`, right?

Let's see.

```
$ docker inspect -f "{% raw %}{{ .Size }}{% endraw %}" dockerlayers:0.1
169609843
```

```
docker inspect -f "{% raw %}{{ .Size }}{% endraw %}" dockerlayers:0.2
169609843
```

This inspection shows that, even if we removed a file in `dockerlayers:0.2`, actually `dockerlayers:0.1` and `dockerlayers:0.2` have the size. This is because the extra layer hides the file, not removes it.

Now, let's create a file named `index.html`:
```
<html>
  <head>
    <title>Docker layers</title>
  </head>
  <body>
    <h1>Docker layers</h1>
    <p>This is an example showing Docker layers</p>
  </body>
</html>
```

Let's modify the `Dockerfile`:
```
FROM httpd:2.4
RUN apt update -y && apt upgrade -y && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists*
RUN rm -f /usr/local/apache2/htdocs/index.html
WORKDIR /usr/local/apache2/htdocs
COPY index.html .
```

And let's build `dockerlayers:0.3`: `docker build -t dockerlayers:0.3 .`

If we inspect the layers of `dockerlayers:0.3`, we can see that `dockerlayers:0.3` consists of the layers of `dockerlayers:0.2`, plus an extra layer:
```
$ docker inspect -f "{% raw %}{{ range .RootFS.Layers }}{{ println . }}{{end}}{% endraw %}" dockerlayers:0.3
sha256:ed7b0ef3bf5bbec74379c3ae3d5339e666a314223e863c70644f7522a7527461
sha256:568467bc0db529f94b52d3cfff6110cfdaf996fac92e27e0a00d052a644f6e8b
sha256:16f81c3ee33ae40dac0f3ffa6771829eda3ea2b6f3ded534cc1b3b869a822272
sha256:e7bd853ca2e701ab964af00be463fc2a62f4c34a913deda78d90f4903c57627e
sha256:355053d0995ea7154219a87a700d654fcabf80d8586c52f8a19d5a036607413d
sha256:0fab92a4da126eed2f6a50999be862d90baa8696a89ecee20fcd4dece5bd77c4
sha256:b9e6b9e3cb0a7a4a1e1796eba3bb3f6c3c5822caa395f356ddb6bb4417e042f7
sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef
sha256:6694fb6a6c63b504502e3bb0f2e29ad8d4625f1c3adc5d9f68ccc6fe5db01652
```

We can see that the size of `dockerlayers:0.3` is bigger than the size of `dockerlayers:0.2`, since the extra layer adds a new file.
```
$ docker inspect -f "{% raw %}{{ .Size }}{% endraw %}" dockerlayers:0.3
169610007
```

This example shows how a Docker image has multiple layers, and how each Docker instruction creates an additional layer. 

## The R/W layer

Now, that we observed that a Docker image consists of multiple layers, let's turn our attention to Docker containers.
A Docker container is built from a Docker image. When a Docker container is created, a new layer is added, called the R/W layer.

![Project structure]({{ "/assets/2023-04-30-navigating-docker-commands/rw_layer_one_container.png" | absolute_url }})

All changes made in the container, like adding new files, deleting files, are written in the R/W layer. Once the container is removed, the R/W layer (and all the changes mad ein that container) are deleted. 
To make data persistent across container deletions, Docker volumes should be used.

Since each container has its own R/W layer where data changes are persistent, multiple Docker containers can be created from the same Docker image, and each container can run independently of the other.

![Project structure]({{ "/assets/2023-04-30-navigating-docker-commands/rw_layer_multiple_containers.png" | absolute_url }})

## Conclusions
1. A Docker image consists of multiple layers.
2. It's not a good idea to add update instructions in a Dockerfile, since the image size will increase unnecessarily. It's better to always use latest Docker image versions. 
3. Once a container is started, a new layer, the R/W layer, is created. The R/W layer is the place where all the changes made in that container are persisted.

### Links

https://learn.acloud.guru/handson/ff4295a9-c1a8-45c7-8c8b-47681b909434

https://learn.acloud.guru/course/06875cfa-fb88-4b28-ac90-150e73bb5a85

https://docs.docker.com/storage/storagedriver/

https://charith.xyz/docker/dockerfile-layered-architecture/
