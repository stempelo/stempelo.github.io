---
title:  'Docker notes'
permalink: '/pages/Docker.html'
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
Command:
    docker build -t example/nano
### Basic Dockerfile example 3
Adding a file
Start from image create in example 2
    FROM example/nano
    ADD note.txt /note.txt
    CMD "nano" "/note.txt"
Create a file note.txt...
Build the image:
    docker build -t example/note .
## syntax of a Dockerfile
### FROM statement
- Which image to download from repo and start from
- must be the first command in a Dockerfile
### MAINTAINER statement
- allows you to set the Author field of the generated images
### RUN statement
- runs the command line (ex: RUN unzip install.zip /opt/installdir)
  - wait for the command to finish
  - save the result
### ADD statement
- adds:
  - local files (ADD note.txt /note.txt)
  - the contents of tar archives
    - ADD archive.tar.gz /installdir/ (automatically uncompresses tar files)
  - works with URLs as well (ADD https://.../download/project.zip /installdir/)
### ENV statement
- sets environment variables
- both during the build and when running the result image. Those enviroment variables will still be set
  - ENV DB_HOST=db_production.example.com
### ENTRYPOINT and CMD statements
- ENTRYPOINT specifies the start of the command to run
  - you can use if your container acts like a command-line program
- CMD specifies the whole command to run
**Shell Form and Exec Form**
- Shell form:
  - nano note.txt
- Exec form:
  - ["/bin/nano", "note.txt"]
### EXPOSE statement
- maps a port into the container
  - EXPOSE 8080 (same thing oh -p 8080:8080)
### VOLUME statement
- defines shared volumes
- define ephemeral volumes
- VOLUME ["/host/path/" "/container/path/"]
- VOLUME ["/shared-data"]
- **Avoid** defining shared folders with your host in Dockerfiles
### WORKDIR statement
- sets the directory the container starts in (WORKDIR /install/)
### USER statement
- sets which user the container will run as (USER bill)
# Images build with multi-project Dockerfiles
- multi-stage builds
Make a Dockerfile
    view folder multi-stage
    docekr run docker build -t google-size .
Run the image in a container
    docekr run google-size .
Check the new file Dockerfile
Then rebuild and check the size of the images
# What Docker Does
- program written in Go
- Docker is a program which manages several features of the kernel
  - uses *cgroups* (control groups) to contain processes
    - group processes together
    - give them the idea of being contained in thei own world
  - uses *namespaces* to contain networks
  - uses *copy-on-write* filesystems to build images
  - makes scripting distributed systems **easy**
# Docker Control Socekt
**Docker is divided into two programs**
- client
- server
  - the server receives commands over a socket
    - network
    - *file*
- Docker uses bridges to create virtual networks inside your host
    docker run -ti --rm --net=host ubuntu:16.04 bash
Inside the container:
- apt-get install
- apt-get install bridge-utils
- brctl show
In another terminal:
    docker network create new-netork
In this example i turn off this protection by passing the --net=host option
## Routing
How docker moves packets between networks and containers
- create *firewall* rules to move packets between networks (iptables commanad)
- NAT
- check
  - sudo iptables -n -L -t nat
    docker run --rm -ti --net=host --privileged=true ubuntu bash
Then inside the container:
- apt-get update
- apt-get install iptables
- iptables -n -L -t nat
In another  terminal start a second container
    docker run -ti --rm -p 8080:8080 ubuntu bash
Return to the first container and rerun:
    iptables -n -L -t nat
You can see the *port forwarding* rule added
## Namespaces
- namespaces are a feature in the Linux kernel that allows you to provide complete network isolation to different processes
**Virtual networking**
Processes --> virtual network devices --> bridges

These private networks are bridged into a shared network with all the containers
- Containers get their own copy oh the networking stack
# Processes and cgroups
- in Docker, a container starts with an init process, runs commands and runs other processes. When this init process exit that container just vanishes
## inspect command
Run a container:
    docker run -ti --rm --name killme ubuntu bash
In a new terminal:
    docker inspect --format '{{.State.Pid}}' killme
Start a super privileged container with:
    docker run -ti --rm --privileged=true --pid=host ubuntu bash
Inside this second container kill the process id of killme container.
kill *pid*
# Limit resource on container
- CPU time
- memory allocation
- inherited limitations and quotas
# Storage
- hardware storage devices
  - logical storage devices (partition drives into groups)
    - Filesystems
      - FUSE filesustems
      - network filesystems
        - on Docker: **COWs** (copy on write)
## Moving Cows
- The contents of layers are moved between containers in gzip files
- Containers are indipendent of the storage engine
**Any container can be loaded anywhere**
# Volumes and Bind mounting
- use the Linux VFS (Virtual File System)
You can:
- mount devices on the VFS
- mount directories on the VFS
- **getting hte mount order correct**
- always mounts the host's filesystem over the guest
# Docker Registry
- is a program
- stores:
  - layers
  - images
  - tag
  - meta data around the images
  - listen on port 5000 (usually)
  - maintains an index and searches tags
## Docker registry programs
- the official Python Docker Registry
- Nexus
## Running the Docker registry in a docker container
In command prompt:
    docker run -d -p 10000:5000 --restart=always --name registry registry:2
Now:
    docker tag ubuntu:14.04 localhost:10000/mylocal/local-ubuntu:99
    docker push localhost:10000/mylocal/local-ubuntu:99
**you must setup authentication and security on this registry before expose it to any network**
## Saving and Loading containers
- docker save
- docker load
Save most important images into an archive
    docker save -o my-images.tar.gz debian:sid busybox ubuntu:14.04
Remove docker images
    docker rmi debian:sid busybox ubuntu:14.04
Load the images archive:
    docker load -i my-images.tar.gz
Usefull for:
- migrating between storage types
- shipping images on disks
# Orchestration systems for Docker
Orchestration systems:
- start your containers
- keep them running
- restart them if they fail
- allow containers to find each other
- resource allocation
## Many alternatives
- Docker compose
  - single machine coordination is the de facto standard
    - designed for testing
    - development
    - staging
    - not for things that scale dynamically
  - For lerger systems
    - Kubernetes
      - containers which tun programs
      - Pods -> group containers that are intended to be run together
      - Services make pods available to others
      - Powerfull system of labels for describing every aspect of your system
      - very flexible system of overlay networking
      - runs equally well on your host or a cloud provider
Get started http://kubernetes.io
Amazon EC2 Container Service (ECS)
AWS Fargate from Amazon (more automated version of AWS ECS)
Amazon Elastik Kubernetes Service (EKS)
Docker Swarm
Google offers its Kubernetes engine
Microsoft offers the Azure Kubernetes Service (AKS)

  - 
- 





 




