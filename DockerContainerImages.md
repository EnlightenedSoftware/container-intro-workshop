## Docker Container Images

### Dockerfile

Example:

```
FROM adoptopenjdk/openjdk8:x86_64-alpine-jdk8u292-b10-slim
#Copy the local git folder docker-config in de /app folder
ADD thorntail-config /app
ADD ./target/toegangsrechten-pdp-webservice*thorntail.jar /app/toegangsrechten-pdp-webservice.jar
WORKDIR /app
ENTRYPOINT ["java", "-jar", "toegangsrechten-pdp-webservice.jar", "-s", "application.yml"]
```

### Usage

#### Inheritance

**FROM**
With _FROM_ you can specify from which existing image the dockerfile should inherit/should be based on.

**ADD**
Copy a file to the container VM image.

**WORKDIR**
Change directory to a specified folder

**ENTRYPOINT**
The main container process that should be run in the container instance.

**RUN**
Run a command in the container. For example:

```
RUN addgroup --gid 1001 -S $SERVICE_NAME && \
    adduser -G $SERVICE_NAME --shell /bin/false --disabled-password -H --uid 1001 $SERVICE_NAME && \
    mkdir -p /var/log/$SERVICE_NAME && \
    chown $SERVICE_NAME:$SERVICE_NAME /var/log/$SERVICE_NAME
```

**USER**

Change user (Note: user must exist!)

#### Links

[Dockerfile cheat sheet](https://devhints.io/dockerfile)

[Official docs](https://docs.docker.com/engine/reference/builder/)

Public Docker Image Repository: [hub.docker.com](https://hub.docker.com/search?type=image)
