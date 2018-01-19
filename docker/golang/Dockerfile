FROM ubuntu:14.04
MAINTAINER hxz <hxzqlh@gmail.com>

ENV USER hxz
ENV ETC  /root/etc
ENV PKG /root/pkg

RUN useradd $USER -d /home/$USER -G sudo -s /bin/bash -m -k -U && \
    echo $USER:123 | chpasswd 

COPY Dockerfile /root/
COPY etc $ETC
COPY pkg $PKG

RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak && \
    cp $ETC/sources.list.ubuntu.14.04 /etc/apt/sources.list 

RUN apt-get update && apt-get install --no-install-recommends -y \
    sudo \
    aptitude \
    vim \
    wget \
    curl \
    jq \
    git \
    make

RUN cp $ETC/.vimrc /root/.vimrc && \
    cp $ETC/.vimrc /home/$USER/.vimrc 

#RUN wget --no-check-certificate https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz 2>/dev/null && \
#    tar xf go1.8.3.linux-amd64.tar.gz -C /usr/local/ && \
#    rm -rf go1.8.3.linux-amd64.tar.gz 

RUN tar xf $PKG/go1.7.6.linux-amd64.tar.gz -C /usr/local/ 

ENV GOROOT /usr/local/go
ENV GOPATH /home/$USER/gop
ENV PATH $GOROOT/bin:$GOPATH/bin:$PATH

RUN mkdir -p $GOPATH/src $GOPATH/lib $GOPATH/bin $GOPATH/pkg && \
    chown -R $USER: /home/$USER && \
    chmod -R 755 /home/$USER && \
    rm -rf $ETC && \
    rm -rf $PKG && \
    rm -rf /var/lib/apt/lists/*

USER $USER
WORKDIR /home/$USER

VOLUME ["$GOPATH/src"]

EXPOSE 46656 46657
