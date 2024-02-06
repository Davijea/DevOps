# DevOps
## 1.1 Document your database container essentials: commands and Dockerfile.
### Postgres container :
sudo docker run --name david-pg -it -p 5432:5432 -d -v /home/ubuntu/Documents/docker_volumes/postgres:/var/lib/postgresql/data --network app-network -e POSTGRES_PWD=david david/postgres

### adminer container :
sudo docker run --name davidadminer --network app-network -p 8080:8080 -d adminer

### Docker file :
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

## 1-2 Why do we need a multistage build? And explain each step of this dockerfile.
Nous avons besoin de different etages pour sequencées les actions effectuées par le conteneur.
Dans notre cas nous compilons le scrit java avant de l'executer.

FROM maven:3.8.6-amazoncorretto-17 AS myapp-build => Depuis l'image maven:3.8.6-amazoncorretto-17 \
ENV MYAPP_HOME /opt/myapp => On defini notre environnement de travail sur /opt/myapp \
WORKDIR $MYAPP_HOME => On defini notre variable de travail sur /opt/myapp \
COPY pom.xml . => On copie le pom.xml de la racine d'ou est le dockerfile sur /opt/myapp \
COPY src ./src \
RUN mvn package -DskipTests => Permet la compilation \

FROM amazoncorretto:17 \
ENV MYAPP_HOME /opt/myapp => On defini notre variable de travail sur /opt/myapp \
WORKDIR $MYAPP_HOME => On defini notre environnement de travail sur /opt/myapp \
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar => Recuperation du script java compilé \

ENTRYPOINT java -jar myapp.jar => Execution du script java \

## 1-3 Document docker-compose most important commands. 1-4 Document your docker-compose file.
### Docker-compose:
version: '3'

services:  \
  david-pg: \
    image: david/postgres \
    container_name: david-pg \
    ports: \
      - "5432:5432" \
    volumes: \
      - /home/ubuntu/Documents/docker_volumes/postgres:/var/lib/postgresql/data \
    networks: \
      - app-network \
    environment: \
      - POSTGRES_PWD=david \

  david-api: \
    image: david/buildjava2 \
    container_name: david-api \
    ports: \
      - "8081:8080" \
    networks: \
      - app-network \
    depends_on: \
      - david-pg \

  david-http: \
    image: david/buildhtml \
    container_name: david-http \
    ports: \
      - "8084:80" \
    networks: \
      - app-network \
    depends_on: \
      - david-pg \

networks: \
  app-network: \
    driver: bridge \

