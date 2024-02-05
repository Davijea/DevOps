# DevOps
## 1.1
Postgres container :
sudo docker run --name david-pg -it -p 5432:5432 -d -v /home/ubuntu/Documents/docker_volumes/postgres:/var/lib/postgresql/data --network app-network -e POSTGRES_PWD=david david/postgres

adminer container :
sudo docker run --name davidadminer --network app-network -p 8080:8080 -d adminer

Docker file :
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
