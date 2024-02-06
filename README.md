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
   
ENV SQL_SCRIPTS /docker-entrypoint-initdb.d \
WORKDIR $SQL_SCRIPTS \
COPY 01-CreateScheme.sql \
COPY 02-InsertData.sql

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
### Commandes les plus utiles du docker-compose
depends_on => Permet de forcer le demarrage d'un conteneur avant un autre
ports => Permet le mapping de ports
volumes => Permet le mapping de volumes
networks => Permet de choisir un reseau
### Docker-compose:
version: '3'

services:
  david-pg:
    image: david/postgres
    container_name: david-pg
    volumes:
      - /home/ubuntu/Documents/docker_volumes/postgres:/var/lib/postgresql/data
    networks:
      - app-network
    environment:
      - POSTGRES_PWD=david
      - POSTGRES_USER=david
      - POSTGRES_DB=davidpg

  david-api:
    image: david/buildjava2
    container_name: david-api
    networks:
      - app-network
    depends_on:
      - david-pg
    environment:
      - POSTGRES_PASSWORD=david
      - POSTGRES_USER=david
      - POSTGRES_URL=david-pg:5432
      - POSTGRES_DB=davidpg


  david-http:
    image: david/buildhtml
    container_name: david-http
    ports:
      - "8084:80"
    networks:
      - app-network
    depends_on:
      - david-pg

networks:
  app-network:
    driver: bridge

## 1-5 Document your publication commands and published images in dockerhub.
### Repo backend :
https://hub.docker.com/r/davijea/backend
### Repo API :
https://hub.docker.com/r/davijea/api
### Repo BDD :
https://hub.docker.com/r/davijea/postgres

## 2-1 What are testcontainers?
C'est comme une boîte à outils pour les développeurs qui permet de tester les logiciel rapidement. Au lieu de configurer manuellement des bases de données ou d'autres services pour les tests, Testcontainers utilise des conteneurs Docker pour les exécuter automatiquement. Ca rend les tests plus fiables et plus rapides, car ils peuvent être effectués dans un environnement iso-production. De plus, ça permet de détecter les problèmes plus tôt et donc d'augmenter l'efficacité de la CI.

## 2-2 Document your Github Actions configurations
name: CI devops 2023
on:
  push:
    branches: #Selection de la branche main et develop
      - main
      - develop
  pull_request:

jobs:
  test-backend: #version de l'OS
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0

      - name: Set up JDK 17 #Version du JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'

      - name: Build and test with Maven
        run: mvn clean verify --file simple-api-student-main/pom.xml #Execution du test à partir du pom.xml
test
