# Using Docker

#### Catalogue:
- [Create & Run Container - Long Way](#creatingrunning-a-docker-container-long-way)
- [Create & Run Container - Short Way](#creatingrunning-a-docker-container-short-way)
- [Create Dockerfiles](#creating-dockerfiles)
- [Create/Run Container using Dockerfile](#creatingrunning-a-docker-container-using-dockerfile)
- [Interact with Container](#interacting-with-containers)
- [Bind ports to Container](#binding-ports-to-container)
- [Save data from Container](#saving-data-from-container)
- [Push image to Docker Hub](#pushing-images-to-docker-hub)

<p align="justify">Containers are created from container images. Container images are compressed & pre-packaged file system that contains your app along with its enviroment and configuration with an instruction on how to start the application. That instruction is called the entry point. If an image doesn't exist locally, docker tries to retrieve it from a container image registry. By default, docker always tries to pull from Docker Hub. The most popular way to interact with Docker Containers is Docker CLI (Command Line Interface).</p>

#### Creating/Running a Docker Container (Long Way):
- Open Powershell and run the commands below.
- `docker container create hello-world:linux # SPECIFIES LINUX VERSION OF IMAGE`
- `docker ps # LIST CONTAINERS ACTIVELY RUNNING`
- `docker ps -all`, `docker ps -a`
- `docker container start [CONTAINER_ID] # EXECUTES THE ENTRY POINT`
- `docker logs CONTAINER_ID # DISPLAY LOG MESSAGES`
- `docker container start --attach CONTAINER_ID # ATTACHES TERMINAL TO CONTAINER OUTPUT`
- `docker --help`, `docker network create --help # EVERY DOCKER COMMANDS ACCEPTS HELP FLAG`
- In create command above, `hello-world` is the image & `:linux` is an Image Tag. 
- Any other Container Exit Code except `0` means there was a problem. See `docker ps`
- Docker can identify a container by the first three/few characters of Container ID.
- [Back to Top](#catalogue)

#### Creating/Running a Docker Container (Short Way):
- `docker run hello-world:linux`
- `docker logs CONTAINER_ID`
- Docker run command = Docker container create + start + attach
- Docker run doesn't return container id. Run docker ps to check out the id.
- [Back to Top](#catalogue)

#### Creating Dockerfiles:
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
- [Back to Top](#catalogue)
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

#### Creating/Running a Docker Container using Dockerfile:
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
- [Back to Top](#catalogue)

#### Interacting with containers:
- Create `server.bash` & `server.Dockerfile` as well as run `dos2unix entrypoint.bash`
- `docker build -f server.Dockerfile -t first-server .`
- `docker run first-server # Be careful, Containers are not interactive by default`
- `docker ps`, `docker kill CONTAINER_ID # Returns id if successfully force-stoped container`
- `docker run -d first-server # Create+Starts container but doesn't attaches terminal`
- `docker exec CONTAINER_ID date # Run additional commands from running-container`
- `docker exec --interactive --tty CONTAINER_ID bash # Start terminal session within container`
- Docker run attaches terminal to container after it starts it.
- Meaning, containers wont accept keystrokes from our terminal even if we are attached to them.
- Terminal session within container is helful while troubleshooting
- Since we're gonna enter keystrokes inside terminal, we specify the flag `--interactive`
- We use `-tty` because we're stating a shell, so it interacts properly with container's terminal.
- Additionaly, we have to specify the shell that we want to use. E.g., `bash`
- To exit out the shell: `CTRL+D`. Pressing this too many times logs out of the real terminal.
- [Back to Top](#catalogue)

```
#!/bin/bash
#Filename: server.bash
if ! bash --version | grep -q 'version 5'; then
  >&2 echo "ERROR: Bash not installed or not the right version."; exit 1
fi
echo "Server started. Press CTRL-C to stop..."; while true; do sleep 10; done
```
```
# Filename: server.Dockerfile
FROM ubuntu
USER root
COPY ./server.bash /
RUN chmod 755 /server.bash
RUN apt -y update
RUN apt -y install bash
USER nobody
ENTRYPOINT [ "/server.bash" ]
```

#### Stoping & Removing Containers:
- `docker run -d our-server` Run a container.
- `docker stop CONTAINER_ID` Stops containers gracefully. Might take upto 10 minutes.
- `docker stop -t 0 CONTAINER_ID` Immediately stops container. Can result in data loss.
- `docker rm [-f] CONTAINER_ID` Removes containers. `-f` force-removes running containers.
- `docker ps -aq` Returns a list of all container ids.
- `docker ps -aq | xargs docker rm` 'docker rm' is run for each line of output (ids).
- `docker rmi` Removes a docker image.
- `docker images` `docker rmi IMAGE_NAME` `docker rmi -f IMAGE_NAME` '-f' is for force-removal.
- The pipe symbol feeds output from left command to the input of the command on right.
- The `xargs` takes input and passes it as argument of another command.
- Just like force-removing containers, `-f` with `rmi` command can also cause unpredictable behaviours.
- [Back to Top](#catalogue)

#### Binding ports to Container:
- `docker build -t our-web-server -f web-server.Dockerfile .`
- `docker run -d --name our-web-server IMAGE_NAME` Name containers with `--name`
- `docker ps`, `docker logs CONTAINER_NAMEorID` Navigate to the website & check
- `docker rm -f our-web-server` Force remove running container
- `docker run -d --name our-web-server -p 5001:5000 our-web-server`
- `docker ps` There should be a mapped port active at this point
- Normally containers use data & ports within themselves.
- Port Mapping allows to access services running inside a container from oustide of docker.
- Website at *https://localhost:5000* shouldn't be unaccessible before port binding.
- Flag `-p` publishes port `portOnHost:containerExposedPort` or `outside:inside`
- [Back to Top](#catalogue)

#### Saving data from Container:
<span align="justify">Deleting a container, all data saved within that container gets deleted. Volume Mounting, maps a folder on host to a folder on the container. Volume mounting switch: `-v` or `--volume HostFolder:ContainerFolder`. Command `docker run --rm ubuntu` removes container after it exists. `--entrypoint sh` sets to start container with shell `sh` instead of default entrypoint. The `-c` option executes commands from a string; i.e. everything inside the quotes. Using `echo` command & redirection operator (`>`), a string can be written to a file. `cat /tmp/file` reads & prints on console, contents of `/tmp/file`. Mapping a file on host that doesn't exist, will be mapped as directory within container.</span>

- `docker run --rm --entrypoint sh ubuntu -c "echo 'Hello there.' > /tmp/file && cat /tmp/file"`
- `cat /tmp/file` The file gets deleted because the container stopped.
- `docker run --rm --entrypoint sh -v /tmp/container:/tmp ubuntu`  
  `-c "echo 'Hello there.' > /tmp/file && cat /tmp/file"`
- `cat /tmp/container/file`  
- `touch /tmp/change_this_file`
- `docker run --rm --entrypoint sh -v /tmp/change_this_file:/tmp/file ubuntu`  
  `-c "echo 'Hello there.' > /tmp/file && cat /tmp/file"`
- `cat /tmp/change_this_file`
- `docker run --rm --entrypoint sh -v /tmp/this_file_does_not_exist:/tmp/file ubuntu`  
  `-c "echo 'Hello there.' > /tmp/file && cat /tmp/file"`
- `cat /temp/this_file_does_not_exist`
- [Back to Top](#catalogue)

#### Pushing images to Docker Hub:
<p align="justify">A *Container Image Registry* is used for storing and tracking container images. Container image are tracked by their *tags*, which consists of the name of the image, and optionally a colon & its version (`latest` if not provided). Docker Hub (publicly accessible) is the docker client's default container image registry. To push an image to Docker Hub, username and optionally version number is appended with image name. The `latest` automatically tag is appended if no version number is provided.</p>

- Create an account on Docker Hub (Web), then locally run: `docker login`
- `docker tag our-web-server USERNAME/our-web-server:0.0.1` Was successful if no response.
- `docker push USERNAME/our-web-server:0.0.1`
- `docker tag our-web-server USERNAME/our-web-server:0.0.2` Pushing with different tag/version
- `docker push USERNAME/our-web-server:0.0.2`, `docker logout`
- The `tag` command works similar to `mv` in linux, renaming docker images.
- Check list of pushed docker images ( Along with `tags`) at Docker Hub (Web).
- To remove an image from Docker Hub (Web): Settings > Delete Repository
- [Back to Top](#catalogue)
