# Docker Compose Container Orchestration

Docker compose is a container orchestration tool that can manage several container instances for you.

Features:

- Can build Docker images
- Can push images to a container registry
- Can run/manage containers on one machine
- Manage storage
    - bind mounts(mount on a specific host)
    - volumes(creates volume managed by docker)
- Manages networking
    - dns
    - Can create a separate docker network on the host including port mapping

# Usage

## Configuratie

Voor je begint moet je 1 configuratie bestand aanpassen:
De configuratie van de app is te vinden in de thorntail-config/application.yml Vul in dit bestand je persoonlijke
database gebruikersnaam/wachtwoord in.

De configuratie is gebaseerd op de configuratie uit de volgende git repo's:
`git.wpakb.kb.nl/kb-ansible/app-configuration/loadbalanced-wpakb/configuratie-toegangsrechten-aws.git` en
`git.wpakb.kb.nl/kb-ansible/app-configuration/loadbalanced-dmz/configuratie-toegangsrechten-pdp.git`

## Docker-compose

### VPN

Om gebruik te maken van de docker config moet je wel verbonden zijn met vpn want de huidige docker config maakt
verbinding met de oracle database in de O omgeving

### Poorten

Let op, om de standaard config te starten moeten de poorten 8080 en 8443 open zijn.

### Applicatie starten

Om de webservice te starten hoef je alleen het volgende commando uit te voeren:
`docker-compose up --build`
of om de app in de achtergrond te starten:
`docker-compose up -d --build`

### Applicatie stoppen:

Geef een interupt in de terminal met `ctrl-c`

of open een terminal en typ:
`docker-compose down`

### Links

[official docs](https://docs.docker.com/compose/compose-file/)

[docker-compose cheat sheet](https://devhints.io/docker-compose)

### docker-compose.yml examples

#### Thorntail Java app using the host network

```
version: "3.9"
services:
  #container naam/dns record in het docker netwerk(werkt niet bij de huidige network host mode)
  pdp-webservice:
    #gebruik het netwerk van de lokale docker host machine
    network_mode: host
    #Bouw de container op basis van de Dockerfile in de huidige directory
    build:
      context: .
    volumes:
      - ./dev/logs:/var/log
      - ./dev/pdp-collection-reports:/app/pdp-collection-reports
      - ./dev/local-pdp-policy-cache:/app/local-pdp-policy-cache
      #Overwrite/mount jar with downloaded jar
      - ~/Downloads/toegangsrechten-pdp-webservice-1.1.5-thorntail.jar:/app/toegangsrechten-pdp-webservice.jar
      
    entrypoint:
      #DEBUGGING aan, wacht niet standaard op de debugger(suspend=n)
      ["java", "-agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n" , "-jar", "toegangsrechten-pdp-webservice.jar", "-s", "application.yml"]
```

#### Oracle 
```
version: "3.5"
services:
  #Container instance + dns name
  oracle-server:
    image: store/oracle/database-enterprise:12.2.0.1-slim
    volumes:
      - data:/ORCL
    #portmap from host port to containerport hostport:containerport
    ports:
      - 1521:1521
# volumes managed by docker
volumes:
  data:
```

#### Spring boot
```
version: '2'

services:
  app:
    image: 'docker-spring-boot-postgres:latest'
    
    build:
      context: .
    container_name: app
    depends_on:
      - db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/compose-postgres
      - SPRING_DATASOURCE_USERNAME=compose-postgres
      - SPRING_DATASOURCE_PASSWORD=compose-postgres
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
          
  db:
    image: 'postgres:13.1-alpine'
    container_name: db
    environment:
      - POSTGRES_USER=compose-postgres
      - POSTGRES_PASSWORD=compose-postgres
```
[source tutorial](https://www.baeldung.com/spring-boot-postgresql-docker)

```
version: '3.6'
services:
  api-gateway:
    container_name: api-gateway
    image: api-gateway
    #specify to which network the service should exist/have access
    networks:
      - gateway
    ports:
      - 9090:8080
    restart:
      on-failure
networks:
  gateway: {}
```