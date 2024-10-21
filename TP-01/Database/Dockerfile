FROM postgres:14.1-alpine

# Environment variables (not necessary if you run with the -e flag)
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

# Copy your initialization scripts into the container
COPY ./my-init-scripts/ /docker-entrypoint-initdb.d/

