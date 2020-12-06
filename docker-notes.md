# Docker
* A software platform for building and running containers
* A docker image is an image built using docker technology

##  Docker CLI
* Docker CLI is used extensively
###  Popular CLI commands

|Command|Description|
|-|-|
build|Creates container image <br>Requires a docker file <br>Tag
tag| Name images <br> Doesn't overwrite existing images <br> reates new tags that point to images
images|Lists all images, their repositories and tags, and their size
run|Runs a container <br> Suited for testing an image that has been built
push, pull| Stores images in and retrieves images from remote location

## Container runtime
### Container stack:
app1,app2,app3,app4 <br>
Container Runtime (like docker) <br>
Operating System <br>
Hardware <br>

## Development of a container
* A docker file serves as a blueprint for an image that outlines steps to build the image
* An image is an immutable file that contains everything necessary to run an application
* Images are templates for containers
* A container is a running image that. A layer is put on top of read-only image
* Docker file -(build)-> Image -(run)-> Container

## Image layers
* Images are built using instructions in adocker file
* Each docker instructio creates a new read-only layer
* A writeable layer is added when an image is run as a container
* Layers can be shared between images, which saves disk space and network bandwidth
  
## Dockerfile instructions
Instruction | Description
-|-
FROM|Define base image
RUN|Execute arbitrary commands
ENV|Set environment variables
ADD and COPY | Copy files and directorys. (add can also add from remote urls where as copy only copies from local)
CMD|Define default command for container execution

### Example
FROM ubuntu:18.04
COPY ./app
RUN make /app
CMD python /app/app.py

## Container Registries
* Storage and distribution of named container images
* Public or private
* Hosted or self-hosted
* </> ---push---> Container registry ---pull---> Env

### Image naming
* hostname/repository:tag
* example docker.io/ubuntu:18.04

## Example

### Write docker file
* FROM node:9.4.0-alpine
* COPY app.js .
* COPY package.json .
* RUN npm install &&\ app update && \ apk upgrade
* CMD node app.js

### Build image
* docker build -t my-app:v1 .
* -t indicates tag option so we can provide the name to our image
* hostname can ommited if it goes to docker hub
* . uses current working directory as the build context

### Tagging existing image (optional)
* If we want to use this image in another context, we can tag it and use it again
* docker tag my-app:v1 second-app:v1

### Running container
* docker run my-app:v1

### Push to registry
* finally, push the image to a registry sot hat we can contribute it and store it for later use
* docker push my-app:v1