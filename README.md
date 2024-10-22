# DevOps

### Why should we run the container with a flag -e to give the environment variables?

Running a container with the `-e` flag allows you to pass environment variables, which helps configure the container's behavior without modifying its code. This enables dynamic settings like database credentials or API keys during runtime.

---
---

### Why do we need a volume to be attached to our postgres container?

We attach a volume to the PostgreSQL container to persist database data, ensuring it is not lost when the container is stopped or removed. This allows the database to maintain its state across container restarts.

---
---

### Database documentation

#### Dockerfile:
```
FROM postgres:14.1-alpine

# Environment variables (not necessary if you run with the -e flag)
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

# Copy your initialization scripts into the container
COPY ./my-init-scripts/ /docker-entrypoint-initdb.d/
```

#### Essential commands:

`docker build -t <YOUR_USERNAME>/database`

`docker network create app-network`

`docker run --name my-db-container --network app-network -v ${PWD}/postgresql/data:/var/lib/postgresql/data -d <YOUR_USERNAME>/database`

`docker run -p "8090:8080" --net=app-network --name=adminer -d adminer`

---
---

### Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build in Docker separates the build and runtime environments, leading to smaller, more efficient, and secure images. This technique is especially useful when building Java applications that require a JDK for compilation but only need a JRE for execution.

```
# Build

FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
# This uses the Maven 3.8.6 image with Amazon Corretto JDK 17 for building the Java app.
# This stage is named "myapp-build" for later reference in the multistage build.

ENV MYAPP_HOME /opt/myapp
# Defines an environment variable MYAPP_HOME to store the application path.

WORKDIR $MYAPP_HOME
# Sets the working directory inside the container to the directory defined by MYAPP_HOME.

COPY pom.xml .
# Copies the Maven project file (pom.xml) to the working directory in the container.
# This helps with dependency resolution before copying all source files.

COPY src ./src
# Copies the source code directory "src" into the container’s "src" directory.

RUN mvn package -DskipTests
# Runs the Maven command to compile the code and build the package (JAR file).
# The -DskipTests flag skips the tests to speed up the build.

# Run
FROM amazoncorretto:17
# Uses a lightweight image with the Amazon Corretto JRE 17 to run the application.

ENV MYAPP_HOME /opt/myapp
# Reuses the environment variable to store the application path in the runtime container.

WORKDIR $MYAPP_HOME
# Sets the working directory to the same directory as in the build stage.

COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
# Copies the compiled JAR file from the build stage into the runtime stage.
# The "--from=myapp-build" specifies that it's copying from the first build stage.

ENTRYPOINT java -jar myapp.jar
# Defines the command to run the application when the container starts.
# This runs the JAR file using the Java runtime inside the container.
```

---
---

### Why do we need a reverse proxy?

A reverse proxy enhances security by hiding backend servers and controlling client access. It enables load balancing, distributing traffic across multiple servers for better performance and scalability. It can also handle SSL termination, simplifying secure connections. Additionally, reverse proxies offer caching for faster response times and act as a single entry point, simplifying client interactions with multiple backend services.

---
---

### Why is docker-compose so important?

Docker Compose is important because it makes it easy to manage multiple containers at once using just one YAML file. It helps to define all the services, networks, and volumes your app needs. With Compose, you can start or stop all your containers with a single command, making everything faster and simpler. Plus, it ensures that your setup is the same in development and production, so you avoid surprises when you deploy. It’s a huge time-saver for working with complex apps!

---
---

### Document docker-compose most important commands. Document your docker-compose file.

Build the services:
`docker-compose build`

Start the services: 
`docker-compose up`<br>
Starts all the services in the background, builds them if necessary, and attaches the logs.

Stop the services:
`docker-compose stop`<br>
Stops the running services but keeps the containers.

Stop and remove containers, networks, and volumes:
`docker-compose down`

View logs:
`docker-compose logs`

Check the status of services:
`docker-compose ps`

Rebuild and restart the services:
`docker-compose up --build`

---

### Docker-compose Overview

This file defines a multi-container Docker application with three services: `backend`, `database`, and `httpd`, all connected via a custom network `my-network`.

#### 1. Adminer Service

- **image**: Uses the `adminer` image, which provides a database management tool.
- **ports**:
  - Maps port 8090 on the host to port 8080 inside the container, allowing access to the Adminer UI via `http://localhost:8090`.
- **networks**:
  - Connects to the custom network (`my-network`), allowing it to interact with other services (such as the database).

#### 2. Backend Service

- **build**:
  - The context is set to `./simpleapi`, indicating that the `Dockerfile` for the backend is located in the `simpleapi` directory.
- **ports**:
  - Maps port 8080 on the host to port 8080 inside the container for backend API access.
- **environment**:
  - Passes necessary environment variables like `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD` from `.env` or the runtime environment.
- **networks**:
  - Connects to the custom network (`my-network`) for communication with other services.
- **depends_on**:
  - Depends on the `database` service. The `service_healthy` condition ensures that the backend will only start when the database is healthy.

#### 3. Database Service

- **build**:
  - The context is set to `./Database`, indicating that the `Dockerfile` for the database is located in the `Database` directory.
- **environment**:
  - Environment variables `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` are passed, usually coming from `.env` or runtime inputs.
- **ports**:
  - Maps port 5432 on the host to port 5432 inside the container for PostgreSQL database access.
- **networks**:
  - Connects to the custom network (`my-network`).
- **volumes**:
  - A volume is attached to persist database data in `/var/lib/postgresql/data`, ensuring data persistence even when the container is stopped or removed.
- **healthcheck**:
  - The health check ensures the database service is ready by running the command `pg_isready` with user and database parameters. It checks every 10 seconds with a maximum of 5 retries, timing out after 5 seconds.

#### 4. HTTPD (Frontend) Service

- **build**:
  - The context is set to `./Frontend`, indicating that the `Dockerfile` for the frontend is located in the `Frontend` directory.
- **ports**:
  - Maps port 80 on the host to port 80 inside the container, making the frontend available at `http://localhost`.
- **networks**:
  - Connects to the custom network (`my-network`).
- **depends_on**:
  - Depends on the `backend` service, ensuring it starts after the backend is ready.

#### 5. Networks

- **my-network**:
  - Defines a custom bridge network (`my-network`) that connects all the services, allowing communication between them.

#### 6. Volumes

- **database**:
  - A volume is created for the database to ensure persistent storage for PostgreSQL data.

---
---

### How to publish images on Dockerhub?

This section provides the necessary commands to build, tag, and push Docker images to DockerHub.

#### 1. Build and Tag the Docker Images

For each service, you'll need to build and tag the images before pushing them to DockerHub.
```bash
docker build -t <YOUR_DOCKERHUB_USERNAME>/<YOUR_IMAGE_NAME>:1.0 ./<YOUR_REPOSITORY>
```
Or if the image is already built:
```bash
docker tag <YOUR_IMAGE_NAME> ryokenka/<YOUR_IMAGE_NAME>:1.0
```

#### 2. Push the Docker Images to DockerHub

Once the images are built and tagged, push them to your DockerHub repository.

First login:
```bash
docker login
```
Then you can push the image with:
```bash
docker push <YOUR_DOCKERHUB_USERNAME>/<YOUR_IMAGE_NAME>:1.0
```

### Why do we put our images into an online repo?

We put Docker images in an online repo so we can easily share them with teammates and keep things consistent across different computers. It helps us manage different versions of our app, so we can switch between them if needed. The repo also acts as a backup in case something goes wrong locally. Plus, it makes deployment to servers or the cloud simpler. Finally, having the images online with some documentation makes it easier to understand and use later.


### Images for this project:
[Database](https://hub.docker.com/layers/ryokenka/tp-01-database/1.0/images/sha256:0c029ff66357d02d27ecb68442a1370932e1bce215913fbbb31c0d660f1460a5?uuid=D8C6B495-B9D3-40CE-B30D-107ECB100590)<br>
[Backend](https://hub.docker.com/layers/ryokenka/tp-01-backend/1.0/images/sha256:0c9068df49a244f6f116dcd2058ebfca822fb8f1be57e4d6ea3c9fa9b05a9515?uuid=D8C6B495-B9D3-40CE-B30D-107ECB100590)<br>
[Frontend](https://hub.docker.com/layers/ryokenka/tp-01-frontend/1.0/images/sha256:68d3c72af271ac2a7ec3798b73ec74441a3c7ba2ef590995073eee9b50f16e1b?uuid=D8C6B495-B9D3-40CE-B30D-107ECB100590)<br>
