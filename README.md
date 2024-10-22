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

#### Docker-compose.yml Overview

This file defines a multi-container Docker application with three services: `backend`, `database`, and `httpd`, all connected via a custom network `my-network`.

#### 1. Backend Service

- **build:**
  - **context:** The path to the build directory (`./simpleapi`).
  - **dockerfile:** The Dockerfile to use (`Dockerfile`).
- **networks:**
  - Specifies the custom network (`my-network`) for communication with other services.
- **depends_on:**
  - Specifies a dependency on the `database` service and waits until the `database` service is healthy before starting.


#### 2. Database Service

- **build:**
  - **context:** The build context for the database service (`./Database`).
  - **dockerfile:** The Dockerfile to use (`Dockerfile`).
- **networks:**
  - Uses the custom network (`my-network`) to communicate with other services.
- **volumes:**
  - Maps a local directory (`./Database/postgresql/data`) to the PostgreSQL data directory inside the container (`/var/lib/postgresql/data`).
- **healthcheck:**
  - Ensures the service is healthy by using the `pg_isready` command to check database readiness:
    - **test:** Executes a command inside the container to check if PostgreSQL is ready.
    - **interval:** How frequently to perform the health check (every 10 seconds).
    - **retries:** How many times to retry the check before marking the service unhealthy (5 times).
    - **timeout:** How long to wait for each health check (5 seconds).


#### 3. HTTPD (Frontend) Service

- **build:**
  - **context:** The build directory for the frontend service (`./Frontend`).
  - **dockerfile:** The Dockerfile to use (`Dockerfile`).
- **ports:**
  - Maps port 80 on the host to port 80 inside the container.
- **networks:**
  - Connects to the custom network (`my-network`).
- **depends_on:**
  - Specifies a dependency on the `backend` service, ensuring it starts only after the backend is ready.


#### 4. Networks

- **my-network:**
  - Defines a custom bridge network called `my-network` that connects all the services. The `bridge` driver is the default network type, which isolates containers within this network and allows them to communicate with each other.


This setup creates a connected system where:
- `httpd` (the frontend) interacts with `backend`.
- `backend` relies on the `database`.
- The custom network ensures seamless communication between the services.
