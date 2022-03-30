---
layout: post
title:  'Docker training'
date: '2022-03-21'
permalink: '/Docker'
---
# CLI snippets

docker info

docker images

docker run -ti ubuntu:latest bash

In another terminal
    docker ps

All container, running and stopped
docker ps -a

last container stopped
docker ps -l

## The Docker flow
### Images to Containers
docker run
- containers have a main process
- the container stops when that process stops
- containers have names
### Containers to Images
docker commit "container id"

docker tag "xxxxxxx"

With one command:
docker commit "name container" "name image"

# Run processes in containers
example:
docker run --rm -ti ubuntu sleep 5
docker run -ti ubuntu bash -c "sleep 3; echo all done"

## Running container in background
docker run -d -ti ubuntu bash

In another terminal:
- docker ps -l
- docker attach "name container"

## Leaving Processes Running in a Container
### docker attach
- detached containers
- interactive containers
- detach with CTRL-p e CTRL-q
- ...
- docekr attach "container name"

## Running more processes in a container
### docker exec
- starts another process in an existing container

# Manage Containers
## docker logs
Example log:
    docker run --name example -d ubuntu bash -c "lose /etc/password"
Now for check the logs:
    docker logs example

## Stopping containers
Makes it stop
    docker kill "container_name"
## Remove containers
    docker rm "container_name"
# Resource Contraints on container
- memory limits
  - docker run --memory maximum-allowed-memory image_name command
- CPU time limits
  - docker run --cpu-share ... (relative to other containers)
  - docker run --cpu-quota ... (to limit it in general)
# Networking in containers
- by default programs in containers are isolated from the internet
- you can:
  - group containers into "private" networks
  - explicity choose who can connect to whom
  - expose ports to let connections from external to in
## Example echo-server
Container 1
    docker run --rm -ti -p 10000:10000 -p 10001:10001 --name echo-server ubuntu:14.04 bash
Inside container echo-server:
    nc -lp 10000 | nc -lp 10001

Container 2
    docker run --rm -ti ubuntu:14.04 bash
inside the container 2
    On Windows
        nc ip_local_machine 10000
    On Mac use host.docker.internal

Container 3
    docker run --rm -ti ubuntu:14.04 bash
inside the container 3
    On Windows
        nc ip_local_machine 10001
    On Mac use host.docker.internal

Now write something in container 2, it will printed on container 3

## Exposing ports dinamically
- the port inside the container is fixed
- the port on the host is chosen dinamically from the unusued ports
### Example
Container echo-server
    docker run --rm -ti -p 10000 -p 10000 --name echo-server ubuntu:14.04 bash
Inside the container:
    nc -lp 10000 | nc -lp 10001
In another terminal run:
    docker port echo-server
This command print the port on the host

## Not only TCP
- docker run -p outside_port:inside_port/protocol(tcp/udp)
  - ex: docker run -p 1234:1234/udp

# Container networking
    docker network ls
We can create a network
    docker network create exnetwork
Create a container with name "cat" inside the network
    docker run --rm -ti --net exnetwork --name cat ubuntu:14.04 bash
Create a container with name "dog" inside the network
    docker run --rm -ti --net exnetwork --name dog ubuntu:14.04 bash
Now the two containers are on the same network
## Add a container to a network
Suppose you have a network with name networkalfa
Let's create it:
    docker network create networkalfa
Now, add an existing container with name "app1" to the network "networkalfa":
    docker network connect networkalfa app1
## Legacy linking
- links all ports, though only one way
  - legacy linking only operates in one direction
- secrets enviroment variables are shared only one way

docker run --rm -ti -e SECRET=secretone --name sersecret ubuntu:14.04 bash
docker run --rm -ti --link sersecret --name server2 ubuntu:14.04 bash
Now you can test in the container server2 that if you type the command env you can see the enviroment variable definined in the container sersecret

# Listing images - Manage images
    docker images
Lists downloaded images
## Tagging images
- tagging gives images names
    docker commit...
- name structure:
  - registry.example.com:port/organization/image-name:version-tag
    - organization/image-name is often enough
Images can be build very quickly.
For remove images:
    docker rmi image-name:tag
    docker rmi image-id
# Volumes
Sharing data between containers, host or both.
Two types:
- persistent
- ephemeral, not permanent
## Sharing data with the host
    mkdir volumedocker
Use the option -v
docker run -ti -v C:\Users\mboffo\volumedocker:/shared-folder ubuntu bash
## Sharing data between containers
    volumes-from
- shared "discs" that exist only as long as they are being used
- can be shared between containers
    docker run -ti -v /shared-data ubuntu bash
- put some content inside /shared-data
    echo hello > /shared-data/shared-file
- start another container
    docker run -ti --volumes-from "name_container" ubuntu bash
- inside the folder /shared-data you can view the content file
When you exit the content vanish. It's not permanent
# Docker Registries
- Registries manage and distribute images
  - download and upload images from them
  - you can search images
## Finding images
- Docker search command
  - docker search ubuntu
- From commandline:
  - docker login
    - docker pull debian:sid
    - docker tag debian:sid login_name/test-image:v99
    - docker push login_name/test-image:v99
# Build docker images
## Dockerfiles
- small "program" to create an image
    docker build -t "name" .  <--- point (location of your dockerfile)
- the result will be in your local docker registry
- it's not a shell script!
- each line takes the image from the previous line
- caching with each step
  - docker skips lines that have not changed since the last build
    - watch "using cache" on the output
### Basic Dockerfile
    FROM busibox
    RUN echo "building super simple docker image"
    CMD echo "Hi container"
Save in file named: Dockerfile
Build the image:
    docker build -t first_image .
### Basic Dockerfile example 2
Installing a program
    FROM debian:sid
    RUN apt-get -y update
    RUN apt-get -y install nano
    CMD ["/bin/nano", "/tmp/notes"]
### Basic Dockerfile example 3
Adding a file
Start from image create in example 2
