# TP part 02 - Github Actions

## Main.yml

```yml
name: CI devops 2024
on:
  push:
    branches: 
      - main
      - develop 
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - name: Checkout code
        uses: actions/checkout@v2.5.0

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          architecture: 'x64'

     #finally build your app with the latest command
      - name: Build and test simple-api-student with Maven
        run: cd TP01/simple-api-student-main && mvn clean verify

      - name: Build and test simple-api with Maven
        run: cd TP01/simpleapi && mvn clean verify

  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-22.04
    # steps to perform in job
    steps:
      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./simple-api-student-main
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_AUTH}}/tp01-api:latest

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/database
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_AUTH}}/tp01-postgres:latest

      - name: Build image and push http
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/web
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_AUTH}}/tp01-web:latest
```

## The goal of the pom.xml file

The goal of the `pom.xml` file is to define the project's dependencies, plugins, and other configurations. It is an XML file that contains information about the project and configuration details used by Maven to build the project.

## What are testcontainers?

Testcontainers is a Java library that provides anything that can run in a Docker container for use in JUnit tests. It is a Java library that allows you to use Docker containers for testing.

## Secured Variables, why?

Secured variables are used to store sensitive information such as passwords, API keys, and other secrets. They are encrypted and can only be accessed by authorized users.

## Why did we put needs: test-backend on this job? 

We put `needs: test-backend` on this job to ensure that the `build-and-push-docker-image` job only runs after the `test-backend` job has completed successfully. This ensures that the code is compiling and tests are passing before building and pushing the Docker image.

## For what purpose do we need to push docker images?

We need to push Docker images to a registry so that they can be deployed to a production environment. By pushing Docker images to a registry, we can easily deploy the application to different environments and scale it as needed.