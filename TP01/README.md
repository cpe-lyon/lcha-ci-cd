# TP part 01 - Docker

# Database

```Dockerfile
FROM postgres:14.1-alpine

COPY CreateScheme.sql /docker-entrypoint-initdb.d

COPY InsertData.sql /docker-entrypoint-initdb.d

ENV POSTGRES_DB=$POSTGRES_DB \
   POSTGRES_USER=$POSTGRES_USER \
   POSTGRES_PASSWORD=$POSTGRES_PASSWORD
```

`docker build -t louischarnay/postgres .`

`docker run -p 5432:5432 --network tp01-network --name postgres -e POSTGRES_DB='db' -e POSTGRES_USER='usr' -e POSTGRES_PASSWORD='pwd' -v ./database:/var/lib/postgresql/data louischarnay/postgres`

# Adminer

`docker pull adminer`

`docker run --network tp01-network --name adminer -p 8081:8080 adminer`

The database adminer is available at `http://localhost:8081/?pgsql=lcha-ci-cd-postgres&username=usr&db=db&ns=public`

# API

```Dockerfile
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

`docker build -t louischarnay/api .`

`docker run -p 8080:8080 --network tp01-network --name api -e POSTGRES_USER='usr' -e POSTGRES_PASSWORD='pwd' louischarnay/api`

# Web
   
```Dockerfile
FROM httpd:2.4

COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

COPY ./public-html/ /usr/local/apache2/htdocs/
```

`docker build -t louischarnay/web .`

`docker run -p 80:80 --network tp01-network --name web louischarnay/web`

# Docker-compose

```yaml
version: '3.8'

services:
  postgres:
    container_name: postgres
    build:
      context: ./db
      dockerfile: Dockerfile
    networks:
      - tp01-network
    volumes:
      - ./db/database:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=usr
      - POSTGRES_PASSWORD=pwd
  api:
    container_name: api
    build:
      context: ./simple-api-student-main
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    networks:
      - tp01-network
    depends_on:
      - postgres
    environment:
      - POSTGRES_USER=usr
      - POSTGRES_PASSWORD=pwd
  web:
    container_name: web
    build:
      context: ./web
      dockerfile: Dockerfile
    ports:
      - "80:80"
    networks:
      - tp01-network
    depends_on:
      - api

networks:
  tp01-network:
```

# Questions 

## Why do we need a volume to be attached to our postgres container?

We need a volume to be attached to our postgres container to persist the data. If we don't use a volume, the data will be lost when the container is removed.

## Why do we need a multistage build? And explain each step of this dockerfile.

We need a multistage build to separate the build and run environments. The build environment is used to compile the application and the run environment is used to run the application. This allows us to have a smaller image for the run environment.

## Why do we need a reverse proxy?

We need a reverse proxy to route requests to the correct server. It allows us to have a single entry point for all requests and to have a single domain name for all services.

## Document docker-compose most important commands.

- `docker-compose up`: Builds, (re)creates, starts, and attaches to containers for a service.
- `docker-compose down`: Stops and removes containers, networks, volumes, and images created by `up`.
- `docker-compose build`: Builds or rebuilds services.
- `docker-compose start`: Starts existing containers for a service.
- `docker-compose stop`: Stops running containers without removing them.
- `docker-compose ls`: List containers.

## Why is docker-compose so important?

Docker-compose is important because it allows us to define and run multi-container Docker applications. It allows us to define the services, networks, and volumes in a single file and to run them with a single command.

## Why do we put our images into an online repo?

We put our images into an online repo to share them with others and to be able to deploy them on a server.