# Using Docker

<p align="justify">Containers are created from container images. Container images are compressed & pre-packaged file system that contains your app along with its enviroment and configuration with an instruction on how to start the application. That instruction is called the entry point. If an image doesn't exist locally, docker tries to retrieve it from a container image registry. By default, docker always tries to pull from Docker Hub. The most popular way to interact with Docker Containers is Docker CLI (Command Line Interface).</p>

Creating/Running a Docker Container (Long Way):
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

