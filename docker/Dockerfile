##
## Build da pasta pai através do comando:
##  docker build -f docker/Dockerfile .
##

FROM ubuntu:20.04

RUN apt update
RUN apt install -y nginx

ADD ./nginx/sites-enabled/proxy /etc/nginx/sites-enabled/proxy
