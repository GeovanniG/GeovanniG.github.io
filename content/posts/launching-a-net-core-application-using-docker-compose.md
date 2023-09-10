---
title: "Launching a .NET Core Application Using Docker Compose"
date: 2023-09-10T00:00:00-07:00
draft: false
---

In our previous article, we discussed [how to launch a .NET Core application using a DockerFile](/posts/launching-a-net-core-application-using-a-dockerfile). In this article we would like to take this one step further with the use of a Docker Compose file.

## What is a Docker Compose file?

A Docker Compose file is a file for launching a collection of related services. For example, if our application is composed of a web app and caching service such as Redis, we can launch a container for each service. Technically, Docker does allow multiple services within a single container, however, according to their [best practices](https://docs.docker.com/config/containers/multi-service_container/), it is recommended to run a service per container.

## Creating a Docker Compose file

A Docker Compose file can run a single or many services. In other words, without much extra effort, we can use our previous DockerFile and launch it using Docker Compose instead. We can do this by creating a `docker-compose.yml` file and adding the following content:

```yml
version: '3.4'

services:
  web:
    image: ${DOCKER_REGISTRY-}web
    build:
      context: .
      dockerfile: <relative-path-to-dockerfile>/Dockerfile
    ports:
      - "80"
      - "443"
    environment:
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_ENVIRONMENT=Development
```

The Docker Compose file starts off with the `version`. The version indicates the format and features supported by the Docker Compose configuration file.
Next, we specify the `services` section, it is here where we list our services, with each having it's own unique name. In most cases, the `image` property is required. In our case since our image is based off of a DockerFile, it is not necessary. Regardless, we decided to include it for easier name tracking.

We also specified `environment` variables and `ports`. The environment variables indicate to the container which ports our application will be listening and available. In our case it will be 443 and 80. We then expose those ports so the host can bind to these ports.

With our Docker Compose file defined, we are ready to start our container. Navigating to the `docker-compose.yml` file and running

```bash
docker-compose up
```

will launch all services within the Docker Compose file. Docker will automatically bind ports to 443 and 80. Using these ports, we can access our application.

To gracefully remove the service we can execute

```bash
docker-compose down
```

## A More Practical Example

The above example given is very basic. Nevertheless, it does what it is meant to do, but it can be improved. For example, the above will expose our application in whichever ports Docker chooses to bind to 443 and 80. Instead we can bind to the ports explicitly using the `ports` property. Furthermore, our .NET application will need access to an external database. We can specify the connection string as an environment variable using the `environment` property. This will override any variable we have in `appsettings.json`.

Putting this all together we end up with the following yml file:

```yml
version: '3.4'

services:
  web:
    image: ${DOCKER_REGISTRY-}web
    build:
      context: .
      dockerfile: <relative-path-to-dockerfile>/Dockerfile
    ports:
      - 63608:443
      - 63607:80
    environment:
      - "ConnectionStrings__DefaultConnection=Server=host.docker.internal;Port=5432;Database=<database>;User Id=<userId>;Password=<password>"
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_ENVIRONMENT=Development
```

Note: Since our database is running on localhost and Docker containers do not have access to localhost, we specify `host.docker.internal` instead. This variable allows Docker containers to access to our locally running applications.

As more services are added to your Docker Compose file, you may need to allow communication or share data between them. This can be accomplished using the `network` and `volume` properties, respectively.

Thanks for reading! 

Also, have a look at Docker's tutorial: [Try Docker Compose](https://docs.docker.com/compose/gettingstarted/)