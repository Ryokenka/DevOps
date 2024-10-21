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