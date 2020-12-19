## Initial build
    docker build -t getting-started .

## Initial run
    docker run -dp 3000:3000 getting-started

## Removing container
### 0. Find container id
    docker ps
### 1a Stop container
    docker stop <container-id>
    docker rm <container-id>
### 1b Alternatively do both in one command
    docker rm -f <container-id>

## Push remote
### give image a new name with docker tag command
    docker tag getting-started ylin36/getting-started
### may need to login to docker first
    docker login -u ylin36
### push
    docker push ylin36/getting-started

## Running new instance with play with docker
https://labs.play-with-docker.com/
add new instance
docker run -dp 3000:3000 ylin36/getting-started

## Example showing how data doesn't persist.
### Start ubuntu cointainer that create a file named /data.txt with a random number beteen 1 and 10000
    docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
### Run in CLI
    cat /data.txt
### Or run in cmd
docker exec <container-id> cat /data.txt
### Run another one and notice there's no file there
    docker run -it ubuntu ls /

## Example showing how to persist volume
### Container named volumes
by default, todo app stores its data in /etc/todos/todo.db
mount the drive and see the data persist
### Create volume
    docker volume create todo-db
    docker rm -f <container-id>
    docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
### Add some items then relaunch
    docker rm -f <container-id>
    docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started

## Diving into mount
To check where docker is actually storing data in named volume
```
    docker volume inspect todo-db
```
```
[
    {
        "CreatedAt": "2020-12-18T21:20:32Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```
The mountpoint is the actual location of themount

## Using bind mounts

* Named mounts are good for storing data if we don't care where the data is stored
* Bind mounts control the exact mountpoint on the host.
* With bind mounts, we can mount our source code into the container to let it see code changes, respond, and let us see the changes right away.

||named mount|bind mount|
-|-|-
host location|docker chooses|you control
mount example (using -v)|my-volume:/usr/local/data|/path/to/data:/usr/local/data
populates new volume with container contents|yes|no
supports volume drivers|yes|no

### To run container to support a dev workflow
* Mount source code into the container
* install all dependencies, including the "dev" dependencies
* Start nodemon to watch for filesystem changes

1. make sure no container is running
2. 
```
docker run -dp 3000:3000 -w /app -v "$(pwd):/app" node:12-alpine sh -c "yarn install && yarn run dev"
```
* -w /app => sets the "working directory that the command will run from"
* -v "$(pwd):/app" => bind mount the curent directory from the host in the container into the /app directory
* node:12-alpine - the base image that doesn't have the source code.
* sh -c "yarn install && yarn run dev" - the command that starts a shell and runs yarn install to install all dependencies and then run  yarn run dev. dev script starts nodemon per package.json.
3. watch the logs
```
docker logs -f <container-id>
```
4. Make a change to the app.js
5. Using bind mounts is very common for local dev setup, providing the advantage of dev machine not needing all the build tools and env installed.

## Multi-container Apps
* Each container should do one thing and do it well.
* There's a good chance you'd have to scale APIs and front-ends differently than databases
* Separate containers let you version and update versions in isolation
* While you may use a container for the database locally, you may want to use a managed service for the database in production. You don't want to ship your database engine with your app then.
* Running multiple processes will require a process manager (the container only starts one process), which adds complexity to container startup/shutdown

### Create the network
* two containers can talk to each other if they're on the same network
```
docker network create todo-apps
```
### Start a MySql container and attach it to the network (docker creates named volume automatically if it doesn't exist)
    docker run -d --network todo-app --network-alias mysql -v todo-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7

### Confirm database is up and running
    docker exec -it <container-id> mysql -p
    SHOW DATABASES;

### Connecting to mysql
* Get nicolaka/netshoot on the same network and check that you can see the network aliased (hostname) mysql from earlier that's on the same network
```
docker run -it --network todo-app nicolaka/netshoot
dig mysql
```

## Running app with MySQL 
### passed in the network alias mysql into environment variable MYSQL_HOST
```
    docker run -dp 3000:3000 -w /app -v "$(pwd):/app" --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"

    docker logs <container-id>
```

### add a couple items to the browser and notice the db change
    docker exec -it <container-id> mysql -p todos
    select * from todo_items;

## Docker Compose
### Check docker compose version
    docker-compose version

### stop all containers
    docker rm -f $(docker ps -a -q)

### Run docker-compose file
* by default docker compose automatically creates a network specifically for the application stack
```
docker-compose up -d
```
* Show all logs or individual logs. -f flag "folows" the lo, so it wil give you live output as it's generated
```
docker-compose logs -f
docker-compose logs -f app
```
Pro tip - Waiting for the DB before starting the app

When the app is starting up, it actually sits and waits for MySQL to be up and ready before trying to connect to it. Docker doesn't have any built-in support to wait for another container to be fully up, running, and ready before starting another container. For Node-based projects, you can use the wait-port dependency. Similar projects exist for other languages/frameworks.

* by default, named volumes are not removed. if you want to remove you will need to do
```
docker-compose down --volumes
```

# Best practices
## Security Scanning
### use Snyk to scan for vulnerbilities
    docker scan getting-started

## Image layering
    docker image history getting-started
    docker image history --no-trunc getting-started

## Layer caching
1.
```
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```
2. create .dockerignore and put in content node_modules
```
FROM node:12-alpine
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --production
COPY . .
CMD ["node", "src/index.js"]
```
