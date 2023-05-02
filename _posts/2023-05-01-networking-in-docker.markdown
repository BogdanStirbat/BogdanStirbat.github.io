---
layout: post
title:  "Networking in Docker"
date:   2023-05-01 12:16:18 +0300
categories: jekyll update
---

Once an application is deployed and runs in [Docker](https://www.docker.com/), it is natural that we want to access it, to send requests and receive responses.
For this reason, networking is a very important concept in Docker.

As mentioned in the [official documentation](https://docs.docker.com/network/), Docker's networking system is pluggable, using drivers. The drivers that exists by itself are:
 - `bridge`, the default network driver. When a network is created, if no driver is specified, this driver will be used. This driver is used for standalone containers that need to communicate.
 - `host`, a driver that offers no isolation between the host network and the container network.
 - `overlay`, for connecting different Docker daemons
 - `ipvlan`, a driver that offers total control of the IPv4 and IPv6 addresses of containers
 - `macvlan`, a driver that allows the assignation of MAC addresses to containers 
 - `none`, a driver that disables all networking.

Once Docker is installed, 3 networks are created by default: `bridge`, `host`, `none`. Each network, with the associated driver, can be reviewed by executing `docker network ls`:
```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
42cb586b0a7f   bridge    bridge    local
c8e8d72c2923   host      host      local
94380546ac7c   none      null      local
```

If a default network is deleted, the only way to bring it back is by re-installing Docker.

To demonstrate these concepts, let's run 3 different scenarios.

#### Scenario 1. A container running in the default `bridge` network.
Let's start a container on the default `bridge` network: `docker run -d --name web_bridge httpd:2.4`

When no network is specified, the default network is `bridge`, so no extra argument for specifying the network was needed. We can inspect that, indeed, the container `web_bridge` is on the network `bridge`:
```
$ docker inspect web_bridge
...
           "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "6ce7e93b7aca6af978bd8c89e30b517534b3260811787219e0f4db94b11f25b9",
                    "EndpointID": "f628854c275058b554e8c9ece5ed5c794e8b99d521afbcbdc1cf8f66b8a608a4",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
...                
```

From the output, it's visible that the container web_bridge is on the network `bridge`, having the IP address 172.17.0.2 .

Now, let's start another container in the `bridge` network, to interact with the `web_bridge` container: `docker run --rm -it busybox`
This newly created container is for test-purposes only, so we want it to be removed after it's executed; this is why `--rm` was added ar command line argument. The `-it` flag specifies that it's an interactive container.

First, let's check the networking of the newly created container: `ip addr`. We can check that the IP address is from the same range as the `web_bridge` container.

Now, let's check the connectivity with the `web_bridge` container:
 - `ping 172.17.0.2`
 - `ping web_bridge`

The experiment shows that we have connectivity to `web_bridge` by IP, since it's in the same network, but not by container name.

The conclusion of this experiment is that different containers in the same default `bridge` network take the IP from the same IP address space, have IP connectivity, but one container name is not recognized in another container.

#### Scenario 2. A container running in a different network, but using the `bridge` driver.

To run this experiment, let's create a new network: `docker network create test`. This new network will use the default `bridge` driver:
```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
6ce7e93b7aca   bridge    bridge    local
4031aca3b7cb   host      host      local
00263dbf022a   none      null      local
a15c55ae210a   test      bridge    local
```

Let's create a new container in the `test` network: `docker run -d --name web_test --network test httpd:2.4`. 

```
$ docker inspect web_test
...
            "Networks": {
                "test": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "d01c54127e19"
                    ],
                    "NetworkID": "a15c55ae210ad7f4ebe00ef529ac2069dcee24214bb65e21dac6aef1567e3a1b",
                    "EndpointID": "13e8dafbc4f665b21cfba40943f794ecbe315cb3da329d298c22915b36b0df78",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:02",
                    "DriverOpts": null
                }
...
```

We can observe that, indeed, the container `web_test` was created in the `test` network. We can also observe the IP address, and the IP address space, of the container `web_test`.

Now, let's create another container in the `test` network: `docker run --rm -it --network test busybox`.

Here, if we run `ip addr`, we can observe that the IP address of the newly created container is in the same IP address space as `web_test`. 

Let's check the connectivity with `web_test`:
 - `ping 172.18.0.2`
 - `ping web_test`

The conclusion is that different containers in the same network are accessible by IP address, and the container name is recognizable in another container form the same network.

#### Scenario 3. A container running in the `host` network

Let's start a container in the `host` network: `docker run -d --name web_host --network host httpd:2.4`.

We can observe that the network of the `web_host` container is shared with the host operating system: `wget localhost` brings the default Apache page, as if we started the Apache daemon on the host operating system, not in a container.

### Conclusions
Containers in the same network take the IP address from the same IP space, and are accessible by the IP address.

Containers that run in the same user-defined network are also accessible by the container name. 

A container on the host network shares the same network as the host operating system's network.

### Links

https://learn.acloud.guru/course/06875cfa-fb88-4b28-ac90-150e73bb5a85/

https://learn.acloud.guru/handson/b172f193-2dec-4690-8f5b-84883ce96a11/

https://docs.docker.com/network/

https://earthly.dev/blog/docker-networking/
