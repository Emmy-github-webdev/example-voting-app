Example Voting App
=========

A simple distributed application running across multiple Docker containers.

Getting started
---------------

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). 


## Linux Containers

The Linux stack uses Python, Node.js, .NET Core (or optionally Java), with Redis for messaging and Postgres for storage.

> If you're using [Docker Desktop on Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows), you can run the Linux version by [switching to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers), or run the Windows containers version.


_Using Docker Run Command_

## Build docker image for voting app

```
example-voting-app$ cd vote
example-voting-app/vote# docker build . -t voting-app
```

Check if the voting-app is created

```
docker images
```

Run the voting-app container on port 5000

```
docker run -p 5000:80 voting-app
```
Run redis container
```
example-voting-app/vote# docker run -d --name=redis redis
docker run -p 5000:80 --link redis:redis voting-app
```
## Build docker image for worker

Run postgress container

```
example-voting-app/vote# docker run -d --name=db postgres:9.4
```
Build worker image

```
example-voting-app/worker# docker build . -t worker-app
```

Run the worker-app container

```
docker run --link redis:redis --link db:db worker-app
```

## Build docker image for result


```
example-voting-app/result# docker build . -t result-app
```
Run the result-app container on port 5001

```
docker run -p 5001:80 --link db:db result-app
```

_User Docker Compose version 1_

- [Install Docker Compose](https://docs.docker.com/compose/install/other/)

- stop all running containers

```
docker stop container_id
dockker ps
```

- Create docker-compose.yml file

```
#cat > docker-compose.yml
redis:
vote:
db:
worker:
result:
```

- Open docker-compose.yml file with vim and edit it

```
redis:
    image: redis
db:
    image: postgres:9.4
vote:
    image: voting-app
    ports:
        - 5000:80
    links:
        - redis

worker:
    image: worker-app
    links:
        - db
        - redis
result:
    image: result-app
    ports:
        5001:80
    links:
        - db
```

Run in this directory:
```
docker-compose up
```


_User Docker Compose version 3_

- [Install Docker Compose](https://docs.docker.com/compose/install/other/)

- stop all running containers

```
docker stop container_id
dockker ps
```

- Create docker-compose.yml file

```
#cat > docker-compose.yml
redis:
vote:
db:
worker:
result:
```

- Open docker-compose.yml file with vim and edit it

```
version: "3"
services:
    redis:
        image: redis
    db:
        image: postgres:9.4
        environment:
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
    vote:
        image: voting-app
        ports:
            - 5000:80

    worker:
        image: worker-app
    result:
        image: result-app
        ports:
            5001:80
```

Run in this directory:
```
docker-compose up
```

### Test

1. First create a redis database container called redis, image redis:alpine.

```
 $ docker run --name redis -d redis:alpine
```
2. Next, create a simple container called clickcounter with the image kodekloud/click-counter, link it to the redis container that we created in the previous task and then expose it on the host port 8085

The clickcounter app run on port 5000.
if you are unsure, check the hints section for the exact commands.

```
 $ docker run -d --name=clickcounter --link redis:redis -p 8085:5000 kodekloud/click-counter
```
3. Let's clean up the actions carried out in previous steps. Delete the redis and the clickcounter containers.

```
docker stop e3700 5fc
docker rm e3700 5fc
```
4. Create a docker-compose.yml file under the directory /root/clickcounter. Once done, run docker-compose up.

The compose file should have the exact specification as follows -

redis service specification - Image name should be redis:alpine.
clickcounter service specification - Image name should be kodekloud/click-counter, app is run on port 5000 and expose it on the host port 8085 in the compose file.

```
services:
  redis:
    image: redis:alpine
  clickcounter:
    image: kodekloud/click-counter
    ports:
    - 8085:5000
version: '3.0'
```

Then run a **docker-compose up -d** command. To run containers in a background, added **-d flag.**

<br>

The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

## Windows Containers

An alternative version of the app uses Windows containers based on Nano Server. This stack runs on .NET Core, using [NATS](https://nats.io) for messaging and [TiDB](https://github.com/pingcap/tidb) for storage.

You can build from source using:

```
docker-compose -f docker-compose-windows.yml build
```

Then run the app using:

```
docker-compose -f docker-compose-windows.yml up -d
```

> Or in a Windows swarm, run `docker stack deploy -c docker-stack-windows.yml vote`

The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).


Run the app in Kubernetes
-------------------------

The folder k8s-specifications contains the yaml specifications of the Voting App's services.

First create the vote namespace

```
$ kubectl create namespace vote
```

Run the following command to create the deployments and services objects:
```
$ kubectl create -f k8s-specifications/
deployment "db" created
service "db" created
deployment "redis" created
service "redis" created
deployment "result" created
service "result" created
deployment "vote" created
service "vote" created
deployment "worker" created
```

The vote interface is then available on port 31000 on each host of the cluster, the result one is available on port 31001.

Architecture
-----

![Architecture diagram](architecture.png)

* A front-end web app in [Python](/vote) or [ASP.NET Core](/vote/dotnet) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) or [NATS](https://hub.docker.com/_/nats/) queue which collects new votes
* A [.NET Core](/worker/src/Worker), [Java](/worker/src/main) or [.NET Core 2.1](/worker/dotnet) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) or [TiDB](https://hub.docker.com/r/dockersamples/tidb/tags/) database backed by a Docker volume
* A [Node.js](/result) or [ASP.NET Core SignalR](/result/dotnet) webapp which shows the results of the voting in real time


Notes
-----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple 
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to 
deal with them in Docker at a basic level. 
