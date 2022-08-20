# Why you should use docker for development

Docker is a tool for building, shipping and running applications. It provides an interface that abstracts away differences in host operating systems and kernel versions, and it's designed to run the same code everywhere. Docker makes it easier to build, ship and run distributed applications by packaging up all of the layers that go into a finished application container.

## Benefit of using docker

Docker containers are typically built from images which are created when users need to package software for distribution. Images contain everything needed to bootstrap a container including dependencies and configuration files. When creating a container from an image, all of the binaries in the image are copied to the container, making them accessible to the application on its host system.

They also helps reduce the risk associated with a new release, this way it makes it safer to attempt a rollback because of its immutable infrastructure (every new release of the application creates a new container)

### setting up your docker environment

Using Docker and docker-compose, you can run a complex application locally on any machine in less than 5 minutes as long as docker and docker-compose is installed on the machine you are using.

to install docker, use the official docker website and follow the instructions
```
sudo apt-get update
```
```
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
```
```
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

## Roadmap to adopting docker

The roadmap below gives each steps of working with containers, for all organizations and teams regardless of their existing knowledge.

* define the architecture
* writing a dockerfile
* writing a docker-compose file
* deploying a docker-compose file
* end to end testing

### define the architecture

Structure the entire application into containers, including the databases.

for standard services like redis, mongoDB, mySQL and many more, we will be able to use standard images from the DockerHub.

Once we have structured all the components for our application we can proceed to the next step.

### writing a dockerfile

There are various ways to write you first Dockerfile, for this example we will be writing a docker file with npm installed to run a node.js application
```
# Choose the image which has node installed already
FROM node:alpine

# COPY all the files from your current project directory into the Container
COPY ./ ./

# Install the project dependencies
RUN npm install

# Open a port 
EXPOSE 80

# Default command to launch the application
CMD ["npm", "start"]
```
we can start using this container image as the official development environment (we can call it the `base image`), and we can begin writing the dockerfiles for the various components of the application using the `base image`

next we deploy the `base image`
```
docker build -t image_name:version .
```

### writing a docker-compose file

A Dockerfile makes it easy to build and run a single container, while a docker-compose file makes it easy to build and run a stack of multiple containers.

```
version: "3.7"
services:
    application:
      container_name: application
      build: application
      dockerfile: Dockerfile
      ports:
        - 3000:80

    hasher:
      container_name: hasher
      build: hasher
      dockerfile: Dockerfile
      ports:
        - 3001:80

    redis:
      container_name: redis
      image: redis

    mongodb:
      container_name: mongodb
      image: mongodb
      ports:
        - 27017:27017
      volume:
        - ~/apps/mongo:/data/db
      environment:
        - MONGO_INITDB_ROOT_USERNAME=username
        - MONGO_INITDB_ROOT_PASSWORD=password

    client:
      container_name: client
      build: client
      dockerfile: Dockerfile
      links:
        - application
        - hasher
        - redis
        - mongodb
```
compose will analyze the docker-compose.yml and pull the required images and build the ones that need to be built.

then it will create a private bridge network for the application and start all the containers in that network.

the private network is necessary to avoid any risk of interference from an external network, also it makes it easier for the containers to communicate with each other since they are in the same network.

### deploying a docker-compose file

deploying our docker-compose file is as simple as typing the command
```
docker-compose up --build
```

### testing the application

Our code needs to be properly tested, for this example we will imagine our application is a node.js application and we will need to test it using mocha.

we will create a folder called `test`
```
mkdir test
```
we will use the code format below and save in the  `./test/test.js` file.
```
var assert = require('assert');
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      assert.equal([1, 2, 3].indexOf(4), -1);
    });
  });
});
```
we can restart and interact with the contianers
```
docker-compose restart
docker exec -it application sh
```
in the container, we can test the application with mocha by running the command
```
./node_modules/mocha/bin/mocha.js
```
if the test pass, we can proceed to adding the test script in the package.json file of the base image
```
{
  ..
  ..
  "scripts": {
    "test": "mocha"
  }
  ..
}
```
update the docker `base image` and restart

```
docker rmi image_id
docker build -t image_name:version .
```
we restart our application container once more so that it can use the update base image
```
docker-compose restart
```
this time, we will run our test with `npm test` command to the containers non-interactively
```
docker exec container_name npm test
```
If needed, you can implement a CI flow to automate the process as part of an end to end testing system.

## Conclusion
The goal of this guide, is to show you how you can use Docker to develop software applications. 

This is ideal for developers because it means they don't have to spend time worrying about how the different parts of their application interrelate and interact with each other - this is taken care of by the container system itself.

although it is certainly a lot of work, but we can get there one step at a time.
