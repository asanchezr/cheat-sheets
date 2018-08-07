# Docker Cheat Sheet <img src="https://github.com/asanchezr/cheat-sheets/blob/master/docker/docker.png" width="50px"/>

<img src="https://github.com/asanchezr/cheat-sheets/blob/master/docker/docker-architecture.png" />

### Useful Commands

For more information, see [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes) and [CodeFresh Everyday Hacks Docker](https://codefresh.io/blog/everyday-hacks-docker/)

List Docker CLI commands
```
docker
docker container --help
```

Display Docker version and info
```
docker --version
docker version
docker info
```

List all the running containers
```
docker ps
```

Download an image
```
docker pull image_name
```

Rename a Docker container
```
docker rename node_v1 node_v2
```

### Delete Commands - Containers

Remove a container and its volume
```
docker rm -v container_name
```

Remove all stopped containers
```
docker rm $(docker ps -aq -f status=exited)
```

Delete all RUNNING and stopped containers  (-f is force)
```
docker rm -f $(docker ps -qa)
```

Remove old containers
```
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```

Stop and remove all containers
```
docker stop $(docker ps -aq) && docker rm $(docker ps -aq)
```

### Delete Commands - Images

Remove an image from the local image store
```
docker rmi image_name
```

Remove dangling images
```
docker rmi $(docker images -q -f dangling=true)
```

Remove all images
```
docker rmi $(docker images -aq)
```

Remove all untagged images
```
docker rmi -f $(docker images | grep "^<none>" | awk "{print $3}")
```

### Delete Commands - Misc

Remove dangling volumes
```
docker volume rm $(docker volume ls -q -f dangling=true)
```

Remove all unused (containers, images, networks and volumes)
```
docker system prune -f
```

Clean all
```
docker system prune -a

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
```

### Build Commands

Build an image from the Dockerfile in the current directory and tag the image
```
docker build -t <tag_name> .
```

### Run Commands

Create and start commands
```
docker run <container_name>:<tag_name>
docker run -p <port_number>:<port_number> <container_name>
docker run -d --name <container_name> -p <port_number> <image_name>:<tag_name> /* Expose container but on a randomly available port */
docker run -d --name <container_name> -p <port_number>:<port_number> <image_name>:<tag_name>
```

Run command will always create a container. Examples of Run command based on the above order:
```
docker run -d redis:latest
docker run -p 3000:80 myapp
docker run -d --name redisDynamic -p 6379 redis:latest
docker run -d --name redisHostPort -p 6379:6379 redis:latest
```

Create and start container, run command
```
docker run -it --name <container_name> <image_name> command
```

Link a container with other containers
```
docker run --name <container_name> --restart always -p <port_name>:<port_name> --link <container_name_1> --net <network_name> --link <hostname> -d <container_name>
```

### Execute Commands

Run a command in a running container
```
docker exec -it <container_name> <command>
```

Example - SSH into a running container
```
# Use `docker ps` to get the name of the existing container
docker exec -it <container_name> /bin/bash
```

Example - using docker-compose
```
docker-compose run <container_name_in_yml> /bin/bash
```

### Logging

For more details about a running container
```
docker inspect <friendly-name|container-id>
```

Display messages the container has written to standard error or standard out.
```
docker logs <friendly-name|container-id>
```

### Dockerfile

```
# Use an official node runtime as a parent image - Define the base image
FROM node:latest

# Create work directory
RUN mkdir -p /usr/src/app

# Set working directory to /usr/src/app
WORKDIR /usr/src/app

# Copy package.json in current directory (host) to work directory (in container)
COPY package.json .

# Install dependencies specified in package.json
RUN npm install

# Copy entire files in current directory (host) to work directory (in container)
COPY . .

# Expose port 3000 for this container
EXPOSE 3000

# Command to start this container
CMD [ "npm", "start" ]
```

### Docker Compose syntax

`docker-compose.yml` file example

```
version: '3'
services:
  app:
    build:
      context: ./docker/app
      dockerfile: Dockerfile
    image: shippingdocker/app:latest
    networks:
     - appnet
    volumes:
     - .:/var/www/html
    ports:
     - ${APP_PORT}:80
    working_dir: /var/www/html
  cache:
    image: redis:alpine
    networks:
     - appnet
    volumes:
     - cachedata:/data
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: homestead
      MYSQL_USER: homestead
      MYSQL_PASSWORD: secret
    ports:
     - ${DB_PORT}:3306
    networks:
     - appnet
    volumes:
     - dbdata:/var/lib/mysql
  node:
    build:
      context: ./docker/node
      dockerfile: Dockerfile
    image: shippingdocker/node:latest
    networks:
     - appnet
    volumes:
     - .:/opt
    working_dir: /opt
    command: echo hi
networks:
  appnet:
    driver: bridge
volumes:
  dbdata:
    driver: local
  cachedata:
    driver: local
```

Create and start containers
```
docker-compose up
```