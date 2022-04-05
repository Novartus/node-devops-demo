## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

### Starting the containers

```bash
$ docker-compose run dev
```
### Test

```bash
# unit tests
$ docker-compose run dev npm run test

# e2e tests
$ docker-compose run dev npm run test:e2e

# test coverage
$ docker-compose run dev npm run test:cov
```

## Docker Setup For Linux

```bash
# Uninstall old versions
$ sudo apt-get remove docker docker-engine docker.io containerd runc

# Install using the repository
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# For x86_64 / amd64
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verify Docker
$ sudo docker run hello-world
```

[Docker Windows](https://docs.docker.com/docker-for-windows/install/) will help you if you are using windows as OS


### Initiating the docker setup for NestJS + MySQL
- First of all we need to create two files namely `Dockerfile` , `docker-compose.yml` in root dir of project. 

- In `Dockerfile` we are going to setup our node backend environment
  
- **NestJS**:
  - Please follow the instructions
```bash 
# We are going to add node:14 as our environment
FROM node:14 AS development # development is the name of environment, you can change it if you want
# Create usr/src/app directory
WORKDIR /usr/src/app # This will create a working dir in docker so our all will be getting started in this folder
COPY  package*.json ./ # It will copy package.json & package-lock.json which are required for npm install
RUN npm install glob rimraf # We have to run this packages as they are in NestJS dependency
RUN npm install --only=development # It will install npm packages
COPY . . # It will copy all code from current repo to docker
RUN npm run build # It will build the package and start the backend
CMD ["node", "dist/main"] # It provide defaults for an executing container
```
- And volla ! Our setup for NestJS is done

- **Database**:
  - First of all we need to config our `docker-compose.yml` file for Database Setup and provide DB dependency to NestJS Container
```bash
version: '3.8' #Docker Compose file version
  services: # We need to define our services here

    dev: # Name of our backend NestJS service
      container_name: nest_backend_dev # Container name
      build: #This will be built during execution
        context: . 
        target: development # Name of our environment specified in `dockerfile`
        dockerfile: ./Dockerfile # Path of the dockerfile
      command: npm run start:debug # Default start command 
      ports: # Port which we are going to use
        - 3000:3000
        - 5050:5050
      depends_on: # This defines the dependency of the service
        - mysqldb # To run this a service called `mysqldb` is required
      env_file: ./.env # Path of environment variable
      environment: # Params of environment variable
        DB_HOST: $DB_HOST
        DB_USER: $DB_USER
        DB_PASS: $DB_PASS
        DB_NAME: $DB_NAME
        DB_PORT: $DB_PORT
      volumes: # Volumes which the container is going to use
        - .:/usr/src/app
        - /usr/src/app/node_modules
      restart: unless-stopped # When it is going to restart


# Now we will add our MySQL database as a service called `mysqldb`
      mysqldb: # Name of our backend DB service
      container_name: nest_database # Container name
      image: mysql # It will use latest mysql image from DockerHub
      restart: always # When it is going to restart
      env_file: ./.env # Path of environment variable
      environment:
        MYSQL_HOST: nest_database # Host name will be the name of docker DB container
        MYSQL_ROOT_PASSWORD: $DB_PASS
        MYSQL_DATABASE: $DB_NAME
      ports: # Port which we are going to use
        - $MYSQL_LOCAL_PORT:$MYSQL_DOCKER_PORT
```
And we are done !