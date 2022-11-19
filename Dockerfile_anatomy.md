# Anatomy of a Dockerfile - Build a Docker Image

* What is a Docker image and a Docker container
* Anatomy of a Dockerfile
* Building a Docker image

## Prerequisites
* Install docker
* a simple web application (we can use Nginx)

## What is a Docker image and a Docker container
A Docker image is a file used to execute code to create a Docker container

A docker container is a super light-weight Virtual machine which has all necessery programs and dependencies in order to run a program or application.

A dockerfile is the configuration setup to create a docker image

## Anatomy of a Dockerfile
Below are some of the basic commands to creating your dockerfile, 

FROM : This command builds an initial layer from an existing image (ever image is based on another image)

WORKDIR: defining the working directory

COPY: copy file from client/local device to the image

ADD: add/copy files from client/local device to the image (similar to COPY)

RUN: run a command during the image build (used for installing dependencies)

CMD: execute a command after the container has been created

ENV: define the environment

EXPOSE: expose a port 

USER: define a user

ENTRYPOINT: define an entrypoint to the container

if you need to go into more complex commands, you can check out the docker reference [here](https://docs.docker.com/engine/reference/builder/)

## Building a docker image
Once our Dockerfile is complete, we can build this with the command
```
docker image build <image-name>:<image-version> .
```

and to run the image
```
docker run <image-name>:<image-version>
```

