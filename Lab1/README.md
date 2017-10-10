# Lab - 1

## Install docker

Open the following link and follow instruction according to your operating system:
[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

## Verify installation

Open the command line and check the version of your set-up:
```
docker --version
```

It should be 17.06+:
```
Docker version 17.06.0-ce, build 02c1d87
```

## Create your first docker machine
`docker-machine` is an utility to create vm including docker on different hosting solutions (e.g. Virtualbox, OpenStack, Amazon). More information here: [https://docs.docker.com/machine/](https://docs.docker.com/machine/). If you have a Linux, you need to install docker-machine.

Create your default VM:
```
docker-machine create --driver virtualbox default
```

Check that the machine is running:
```
$ docker-machine ls
NAME            ACTIVE   DRIVER       STATE     URL                          SWARM   DOCKER        ERRORS
default         *        virtualbox   Running   tcp://192.168.99.100:2376            v17.09.0-ce   
```

Set-up the docker client to use ``default`` machine as environment:
```
eval $(docker-machine env default)
```

## Launch your first container

Run your first application:
``docker run hello-world``

You should have a similar outcome:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
