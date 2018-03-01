FROM ubuntu:16.04
MAINTAINER hxz <hxzqlh@gmail.com>

COPY Dockerfile /root/

RUN apt-get update && apt-get install --no-install-recommends -y \
    aptitude \
    vim \
    && rm -rf /var/lib/apt/lists/*

VOLUME ["/data"]

EXPOSE 80 443
