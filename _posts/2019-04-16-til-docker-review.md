---
layout: post
title:  "TIL - docker review"
date:   2019-04-16
categories: tech
---

# docker stuff
Mostly review, but good to have this for [repetitive memory](https://zedshaw.com/2017/04/24/copying-repetition/).


Docker cheat Sheet: https://github.com/wsargent/docker-cheat-sheet

As root,

Enable and start docker

    systemctl enable docker.service
    systemctl start docker.service

Search for images

    docker search centos

Download image

    docker pull docker.io/jboss/wildfly

Download and run image

    docker run hello-world

View running containers

    docker ps -a

View Console log

    docker logs -f <containerId>

Stop Container

    docker stop <containerId>

Restart container

    docker start <containerId>

Inspect container

    docker inspect <containerId> | less

Changes container's file system

    docker diff <containerId>

Delete container

    docker rm -f <containerId>

Purge all stopped containers

    docker rm $(docker ps -aq)

View downloaded images in local repo

    docker images

Delete image

    docker rmi hello-world

----------------------------------------

# Build customer containers

Example `Dockerfile` which runs a shell script in busybox

{% highlight bash %}                                                            
    # Simple dockerfile with minimal BusyBox base image

    # pulls from docker.io if image not in local repo
    FROM busybox

    MAINTAINER "me"

    # LABEL key-value pairs add metadata
    LABEL version="1.0" \
        description="creates a custom docker image from simple busybox image plus a script"

    # Environment variables
    ENV NAME="Compuglobalhypermeganet" \
        CUSTOM_CONTAINER_DIR="/var/custom-container"

    # excute all following commands as BusyBox root user
    USER root

    # /usr/local/bin/ is in root's $PATH, copy our script to this dir
    COPY my-script.sh /usr/local/bin/my-script.sh

    # make that script executable
    RUN chmod 755 /usr/local/bin/my-script.sh

    # create a dir to use as container mount point
    # shell script will output to local filesystem here
    CMD mkdir $CUSTOM_CONTAINER_DIR
    VOLUME $CUSTOM_CONTAINER_DIR

    # run the script at startup, with $NAME as command line param
    CMD /usr/local/bin/my-script.sh $NAME
{% endhighlight %} 

Make a directory with two files, the `Dockerfile` and `my-script.sh`

### Build the docker image

    docker build . --rm=true -t custom-container

### View it in the local repo

    docker images

### Run it

{% highlight bash %}                                                            
    docker run -d --name="custom" custom-container
    # View logs
    docker logs -f custom
    # Stop it
    docker rm -f custom
{% endhighlight %} 

### Run it with an environment variable
 
    docker run -d --name="custom" custom-container -e NAME="something" custom-container

### View mount points of running container

    docker inspect -f '{{json .Mounts}}' custom | jq

`docker inspect` returns full container definition. `-f {{json .Mounts}}`
selects just the `.Mounts` section as json. `jq` pretty prints it. 

### View output file in host OS

    tail -f /var/lib/docker/volumes/<containerIdHash>/_data/log.log 

### Mount local directory to container

    docker run -d --name="custom-persistent" -v $HOME/some/dir:/var/custom-output custom-container

This maps `$HOME/some/dir/` to `/var/custom-output` in the container. 
`:Z` tells docker to use SELinux permissions.

