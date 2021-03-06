---
description: Swarm release notes
keywords: docker, swarm, clustering, discovery, release, notes
menu:
  main:
    parent: workw_swarm
---

# Install and create a Docker Classic Swarm

You use Docker Swarm to host and schedule a cluster of Docker containers. This section introduces you to Docker Swarm by teaching you how to create a swarm
on your local machine using Docker Machine and VirtualBox.

## Prerequisites

Make sure your local system has VirtualBox installed. If you are using macOS
or Windows and have installed Docker, you should have VirtualBox already
installed.

Using the instructions appropriate to your system architecture, [install Docker
Machine](/machine/install-machine).

## Create a Docker Classic Swarm

Docker Machine gets hosts ready to run Docker containers. Each node in your
Docker Swarm must have access to Docker to pull images and run them in
containers. Docker Machine manages all this provisioning for your swarm.

Before you create a swarm with `docker-machine`, you associate each
node with a discovery service. This example uses the token discovery
service hosted by Docker Hub (only for testing/dev, not for production).
This discovery service associates a token with instances of the Docker
Daemon running on each node. Other discovery service backends such as
`etcd`, `consul`, and `zookeeper` are [available](discovery.md).

1.  List the machines on your system.

    ```bash
    $ docker-machine ls
    NAME         ACTIVE   DRIVER       STATE     URL                         SWARM
    docker-vm    *        virtualbox   Running   tcp://192.168.99.100:2376
    ```

	  This example was run on a macOS system with Docker Toolbox installed. So, the `docker-vm` virtual machine is in the list.

2.  Create a VirtualBox machine called `local` on your system.

    ```bash
    $ docker-machine create -d virtualbox local
    INFO[0000] Creating SSH key...
    INFO[0000] Creating VirtualBox VM...
    INFO[0005] Starting VirtualBox VM...
    INFO[0005] Waiting for VM to start...
    INFO[0050] "local" has been created and is now the active machine.
  	INFO[0050] To point your Docker client at it, run this in your shell: eval "$(docker-machine env local)"
    ```

3.  Load the `local` machine configuration into your shell.

    ```
    $ eval "$(docker-machine env local)"
    ```

4.  Generate a discovery token using the Docker Swarm image.

    The command below runs the `swarm create` command in a container. If you
    haven't got the `swarm:latest` image on your local machine, Docker pulls it
    for you.

    ```console
    $ docker run swarm create

    Unable to find image 'swarm:latest' locally
    latest: Pulling from swarm
    de939d6ed512: Pull complete
    79195899a8a4: Pull complete
    79ad4f2cc8e0: Pull complete
    0db1696be81b: Pull complete
    ae3b6728155e: Pull complete
    57ec2f5f3e06: Pull complete
    73504b2882a3: Already exists
    swarm:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Digest: sha256:aaaf6c18b8be01a75099cc554b4fb372b8ec677ae81764dcdf85470279a61d6f
    Status: Downloaded newer image for swarm:latest
    fe0cc96a72cf04dba8c1c4aa79536ec3
    ```

  	The `swarm create` command returned the `fe0cc96a72cf04dba8c1c4aa79536ec3`
  	token.

  	**Note**: This command relies on Docker Swarm's hosted discovery service. If
  	this service is having issues, this command may fail. In this case, see
  	information on using other types of [discovery backends](discovery.md). Check
  	the [status page](http://status.docker.com/) for service availability.

5.  Save the token in a safe place.

	  You use this token in the next step to create a Docker Swarm.


## Launch the Swarm manager

A single system in your network is known as your Docker Swarm manager. The swarm
manager orchestrates and schedules containers on the entire cluster. The swarm
manager rules a set of agents (also called nodes or Docker nodes).

Swarm agents are responsible for hosting containers. They are regular docker
daemons and you can communicate with them using the Docker Engine API.

In this section, you create a swarm manager and two nodes.

1.  Create a swarm manager under VirtualBox.

    ```
    docker-machine create \
            -d virtualbox \
            --swarm \
            --swarm-master \
            --swarm-discovery token://<TOKEN-FROM-ABOVE> \
            swarm-master
    ```

    For example:

    ```console
    $ docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://fe0cc96a72cf04dba8c1c4aa79536ec3 swarm-master

    INFO[0000] Creating SSH key...
    INFO[0000] Creating VirtualBox VM...
    INFO[0005] Starting VirtualBox VM...
    INFO[0005] Waiting for VM to start...
    INFO[0060] "swarm-master" has been created and is now the active machine.
    INFO[0060] To point your Docker client at it, run this in your shell: eval "$(docker-machine env swarm-master)"
    ```

2.  Open your VirtualBox Manager, it should contain the `local` machine and the
    new `swarm-master` machine.

	  ![VirtualBox](images/virtual-box.png)

3.  Create a swarm node.

    ```bash
    docker-machine create \
    -d virtualbox \
    --swarm \
    --swarm-discovery token://<TOKEN-FROM-ABOVE> \
    swarm-agent-00
    ```

	  For example:

    ```bash
    $ docker-machine create -d virtualbox --swarm --swarm-discovery token://fe0cc96a72cf04dba8c1c4aa79536ec3 swarm-agent-00
    INFO[0000] Creating SSH key...
    INFO[0000] Creating VirtualBox VM...
    INFO[0005] Starting VirtualBox VM...
    INFO[0006] Waiting for VM to start...
    INFO[0066] "swarm-agent-00" has been created and is now the active machine.
    INFO[0066] To point your Docker client at it, run this in your shell: eval "$(docker-machine env swarm-agent-00)"
    ```

3.  Add another agent called `swarm-agent-01`.

    ```bash
    $ docker-machine create -d virtualbox --swarm --swarm-discovery token://fe0cc96a72cf04dba8c1c4aa79536ec3 swarm-agent-01
    ```

	  You should see the two agents in your VirtualBox Manager.

## Direct your swarm

In this step, you connect to the swarm machine, display information related to
your swarm, and start an image on your swarm.


1.  Point your Docker environment to the machine running the swarm master.

    ```bash
    $ eval $(docker-machine env --swarm swarm-master)
    ```


2.  Get information on your new swarm using the `docker` command.

    ```bash
    $ docker info

    Containers: 4
    Strategy: spread
    Filters: affinity, health, constraint, port, dependency
    Nodes: 3
     swarm-agent-00: 192.168.99.105:2376
        ??? Containers: 1
        ??? Reserved CPUs: 0 / 8
        ??? Reserved Memory: 0 B / 1.023 GiB
     swarm-agent-01: 192.168.99.106:2376
        ??? Containers: 1
        ??? Reserved CPUs: 0 / 8
        ??? Reserved Memory: 0 B / 1.023 GiB
     swarm-master: 192.168.99.104:2376
        ??? Containers: 2
        ??? Reserved CPUs: 0 / 8
    ```

    You can see that each agent and the master all have port `2376` exposed. When you create a swarm, you can use any port you like and even different ports on different nodes. Each swarm node runs the swarm agent container.

    The master is running both the swarm manager and a swarm agent container. This isn't recommended in a production environment because it can cause problems with agent failover. However, it is perfectly fine to do this in a learning environment like this one.

3.  Check the images currently running on your swarm.

    ```console
    $ docker ps  -a

    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                     NAMES
    78be991b58d1        swarm:latest        "/swarm join --addr    3 minutes ago       Up 2 minutes        2375/tcp                                  swarm-agent-01/swarm-agent
    da5127e4f0f9        swarm:latest        "/swarm join --addr    6 minutes ago       Up 6 minutes        2375/tcp                                  swarm-agent-00/swarm-agent
    ef395f316c59        swarm:latest        "/swarm join --addr    16 minutes ago      Up 16 minutes       2375/tcp                                  swarm-master/swarm-agent
    45821ca5208e        swarm:latest        "/swarm manage --tls   16 minutes ago      Up 16 minutes       2375/tcp, 192.168.99.104:3376->3376/tcp   swarm-master/swarm-agent-master
    ```

4.  Run the Docker `hello-world` test image on your swarm.

    ```bash
    $ docker run hello-world

    Hello from Docker.
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
            (Assuming it was not already locally available.)
     3. The Docker daemon created a new container from that image which runs the
            executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
            to your terminal.
    ```

    To try something more ambitious, you can run an Ubuntu container with:

    ```bash
    $ docker run -it ubuntu bash
    ```

    For more examples and ideas, visit the [User Guide](/engine/userguide/).

5.  Use the `docker ps` command to find out which node the container ran on.

    ```bash
    $ docker ps -a

    CONTAINER ID        IMAGE                COMMAND                CREATED             STATUS                     PORTS                                     NAMES
    54a8690043dd        hello-world:latest   "/hello"               22 seconds ago      Exited (0) 3 seconds ago                                             swarm-agent-00/modest_goodall
    78be991b58d1        swarm:latest         "/swarm join --addr    5 minutes ago       Up 4 minutes               2375/tcp                                  swarm-agent-01/swarm-agent
    da5127e4f0f9        swarm:latest         "/swarm join --addr    8 minutes ago       Up 8 minutes               2375/tcp                                  swarm-agent-00/swarm-agent
    ef395f316c59        swarm:latest         "/swarm join --addr    18 minutes ago      Up 18 minutes              2375/tcp                                  swarm-master/swarm-agent
    45821ca5208e        swarm:latest         "/swarm manage --tls   18 minutes ago      Up 18 minutes              2375/tcp, 192.168.99.104:3376->3376/tcp   swarm-master/swarm-agent-master
    ```

## Where to go next

At this point, you've installed Docker Swarm by pulling the latest image of
it from Docker Hub. Then, you built and ran a swarm on your local machine
using VirtualBox. If you want, you can onto read an
[overview of Docker Swarm features](index.md). Alternatively, you can develop a
more in-depth view of Swarm by [manually installing Swarm](install-manual.md) on
a network.
