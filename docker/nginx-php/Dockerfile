FROM ubuntu:16.04
MAINTAINER hxz <hxzqlh@gmail.com>

COPY Dockerfile /root/

RUN apt-get update && apt-get install --no-install-recommends -y \
    aptitude \
    vim \
    nginx \
    php7.0 \
    php7.0-fpm \
    php7.0-dev \ 
    && rm -rf /var/lib/apt/lists/*

VOLUME ["/var/www"]

EXPOSE 80 443
