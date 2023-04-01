# Using Docker

<p align="justify">Containers are created from container images. Container images are compressed & pre-packaged file system that contains your app along with its enviroment and configuration with an instruction on how to start the application. That instruction is called the entry point. If an image doesn't exist locally, docker tries to retrieve it from a container image registry. By default, docker always tries to pull from Docker Hub. The most popular way to interact with Docker Containers is Docker CLI (Command Line Interface).</p>

**Creating/Running a Docker Container (Long Way):**
- Open Powershell and run the commands below.
- `docker container create hello-world:linux # SPECIFIES LINUX VERSION OF IMAGE`
- `docker ps # LIST CONTAINERS ACTIVELY RUNNING`
- `docker ps -all`
- `docker container start [CONTAINER_ID] # EXECUTES THE ENTRY POINT`
- `docker logs [CONTAINER_ID] # DISPLAY LOG MESSAGES`
- `docker container start --attach [CONTAINER_ID] # ATTACHES TERMINAL TO CONTAINER OUTPUT`
- `docker --help`, `docker network create --help # EVERY DOCKER COMMANDS ACCEPTS HELP FLAG`
- In create command above, `hello-world` is the image & `:linux` is an Image Tag. 
- Any other Container Exit Code except `0` means there was a problem. See `docker ps`
- Docker can identify a container by the first three/few characters of Container ID.

**Creating/Running a Docker Container (Short Way):**
- `docker run hello-world:linux`
- `docker logs [CONTAINER_ID]`
- Docker run command = Docker container create + start + attach
- Docker run doesn't return container id. Run docker ps to check out the id.

**Creating Dockerfiles:**
- `FROM` instruction specifies the Parent Image from which to build off of.
- `LABEL` instruction is a key-value pair & adds custom meta-data to images.
- `USER` tells which user to use for any dockerfile commands underneath it.
- `COPY` copies files from a directory provided to Docker build command to container image.
- The directory provided to docker build is called context. Usually it is the working directory.
- `RUN` statements are commands to customize the image. 
- This is a great place to install additional softwares or configure requied application files.
- `ENTRYPOINT` specifies the command to run after container has been created from the image.
- *Setting default users for containers created from this image to powerless 'nobody' user*
- *Ensures it can't break out of container & potentially change important files on our host*
```
# Filename: Dockerfile

FROM ubuntu
LABEL maintainer="Abu Muid <dev@abumuid.me>"

USER root
COPY ./entrypoint.bash /
RUN apt -y update
RUN apt -y install curl bash
RUN chmod 755 /entrypoint.bash

USER nobody
ENTRYPOINT [ "/entrypoint.bash" ] # CMD command can be used as well
```
```
#!/bin/bash
# Filename: entrypoint.bash

current_date_time=$(date)
echo "Hello! The current date and time is: $current_date_time"
```

**Creating/Running a Docker Container using Dockerfile:**
- `dos2unix entrypoint.bash` makes windows newline-characters unix compatible.
- `docker build -t first-image .`
- `docker run first-image`
- The docker build command builds an image from a dockerfile.
- Just like containers, images has IDs. `--tag` option associates a convenient name with that ID.
- Docker looks for `Dockerfile` by default. If the name is different use `-f` or `--file` option.
- E.g., `docker build our-first-image [--file or -f] [Dockerfile Name]`
- The context needs to be also specified, which is folder containing files to include in the image.
- Note that, docker images are layer of images compressed together 
- Docker creates an image for every command in Dockerfile, which are called intermediate images.
- After finishing reading Dockerfile, it squashes all the images together into a single image.
