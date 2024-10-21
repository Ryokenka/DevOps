# DevOps

#### Why should we run the container with a flag -e to give the environment variables?

Running a container with the `-e` flag allows you to pass environment variables, which helps configure the container's behavior without modifying its code. This enables dynamic settings like database credentials or API keys during runtime.


#### Why do we need a volume to be attached to our postgres container?

We attach a volume to the PostgreSQL container to persist database data, ensuring it is not lost when the container is stopped or removed. This allows the database to maintain its state across container restarts.


#### Database documentation

Dockerfile:
```
FROM postgres:14.1-alpine

# Environment variables (not necessary if you run with the -e flag)
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

# Copy your initialization scripts into the container
COPY ./my-init-scripts/ /docker-entrypoint-initdb.d/
```

Essential commands:

`docker build -t <YOUR_USERNAME>/database`

`docker network create app-network`

`docker run --name my-db-container --network app-network -v ${PWD}/postgresql/data:/var/lib/postgresql/data -d <YOUR_USERNAME>/database`

`docker run -p "8090:8080" --net=app-network --name=adminer -d adminer`


#### Why do we need a multistage build? And explain each step of this dockerfile.

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


