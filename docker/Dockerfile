FROM debian:jessie
MAINTAINER hxz <huang_xuezhong@vobile.cn>

ENV PKGS                /tmp/vddb_pkg
ENV VDDB                /opt/vdnaserver
ENV VOBILEDATA          /vobiledata
ENV VDNADATA            $VOBILEDATA/vdna
ENV MYSQL_ROOT_PASSWORD 123
ENV MYSQL_USER          vdna
ENV MYSQL_PASSWORD      123
ENV DB_PKGS             /root/vddb_mysql_v5.9.2
ENV ETC                 /home/vdna/etc

# add user
RUN useradd vdna -d /home/vdna -s /bin/bash -m -k -U && \
    echo vdna:123 | chpasswd 

# VDDB && vobiledata
RUN mkdir -p $PKGS $ETC $VDDB \
    $VDNADATA/dna $VDNADATA/dna.tmp $VDNADATA/dna.ds $VDNADATA/dna.pi \
    $VDNADATA/logo $VDNADATA/idna $VDNADATA/idict $VDNADATA/index_dna \
    $VDNADATA/screenshot $VDNADATA/snapshot $VDNADATA/utility_snapshot && \
    chmod -R 777 $VDDB && \
    chown -R vdna:vdna $VDDB && \
    chmod -R 777 $VDNADATA && \
    chown -R vdna:vdna $VDNADATA

COPY targz $PKGS/
COPY etc /home/vdna/etc
COPY entrypoint.sh /usr/local/bin/
COPY Dockerfile /root/

# noninteractive, needn't input password or y... during installing
RUN cp $ETC/sources.list /etc/apt/ && \
    cp $PKGS/init_mysql.sh /root/ && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    mysql-server 

# init mysql && vddb database
RUN bash /root/init_mysql.sh && \
    cd $PKGS && tar xf vddb_mysql_v5.9.2.tgz && \
    cd vddb_mysql_v5.9.2 && \
    mkdir -p $DB_PKGS && mv * $DB_PKGS && \
    bash $DB_PKGS/init_db.sh && \
    sed -i "s/127.0.0.1/0.0.0.0/g" /etc/mysql/my.cnf

RUN dpkg --add-architecture i386 && \
    apt-get update && apt-get install --no-install-recommends -y \
    sudo \
    #aptitude \
    #vim \
    rsyslog \
    nginx \
    apache2 \
    gawk \
    gearman \
    ssl-cert \
    apache2.2-common \
    libapache2-mod-fcgid \
    libfcgi \
    libcgicc5 \
    libapache2-mod-wsgi \
    python-concurrent.futures \
    python-mysqldb \
    python-simplejson \
    python-pip \
    python-dev \
    python-zookeeper \
    python-webpy \
    python-crypto \
    python-setuptools \
    python-netifaces \
    python-pkg-resources \
    python-redis \
    libsnappy1 \
    libssl1.0.0:i386 \
    libmysqlclient18 \
    libmysqlclient18:i386 \
    libglib2.0-0 \
    libglib2.0-0:i386 \
    libstdc++6 \
    libstdc++6:i386 \
    libz1 \
    libz1:i386 \
    libboost-thread1.55.0 \
    libboost-thread1.55.0:i386 \
    liblua5.1-0:i386

RUN cp $ETC/sudoers /etc/sudoers && \
    cp $ETC/vimrc /root/.vimrc && \
    cp $ETC/vimrc /home/vdna/.vimrc && \
    cp $ETC/gearman-job-server /etc/default/gearman-job-server

# deps
RUN cd $PKGS && tar xf vddb_deps_install_v5.9.2.tar.gz && cd vddb5.9.2_deps && \
    if [ ! -d /usr/lib/i386-linux-gnu ]; then mkdir -p /usr/lib/i386-linux-gnu; fi && \
    if [ ! -d /usr/lib/x86_64-linux-gnu ]; then mkdir -p /usr/lib/x86_64-linux-gnu; fi && \
    bash install.sh

# pip-deps
RUN tar xf $PKGS/pip_install.tgz -C /usr/local/lib/python2.7/dist-packages 

# nginx
RUN cd $PKGS && unlink /etc/nginx/sites-enabled/default && \ 
    tar xf nginx_install.tgz && cd nginx_install && \ 
    bash nginx_deploy.sh host.conf

# pyweb
RUN cd $PKGS && tar xf vdna_pyweb_v5.10.1.0_32bit.tar.gz && \
    cd vdna_pyweb_v5.10.1.0 && \
    if [ -e /usr/bin/python2.6 ]; then mv /usr/bin/python2.6 /usr/bin/python2.6.bak; fi && \
    ln -s /usr/bin/python2.7 /usr/bin/python2.6 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/pyweb/* $VDDB/app/pyweb/current/etc/

# webserver
RUN cd $PKGS && tar xf vdna_webserver_v5.9.0.1b_32bit.tar.gz && \
    cd vdna_webserver_v5.9.0.1b && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/webserver/* $VDDB/app/webserver/current/etc/

# apache config
RUN ln -s /opt/vdnaserver/webserver/bin /usr/lib/cgi-bin/vdna && \
    sed -i "s/www-data/vdna/g" /etc/apache2/envvars && \
    sed -i "s/80/88/g" /etc/apache2/ports.conf && \
    chown -R vdna:vdna /var/lib/apache2/fcgid && \
    chown -R vdna:vdna /usr/lib/cgi-bin && \
    cp $ETC/apache2_debian8.000-default /etc/apache2/sites-enabled/000-default.conf

# vddbserver
RUN cd $PKGS && tar xf vdna_vddbserver_v5.7.1.0_32bit.tar.gz && \
    cd vdna_vddbserver_v5.7.1.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/vddbserver/* $VDDB/vddbserver/etc/

# syncagent
RUN cd $PKGS && tar xf vdna_syncagent_ex_v5.9.2.0_32bit.tar.gz && \
    cd vdna_syncagent_ex_v5.9.2.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/syncagent/* $VDDB/syncagent/etc/

# dnapp
RUN cd $PKGS && tar xf vdna_dnapp_zhengcha_v5.7.1.0_32bit.tar.gz && \
    cd vdna_dnapp_zhengcha_v5.7.1.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/dnapp_zhengcha/* $VDDB/dnapp_zhengcha/etc/

# queryweb
RUN cd $PKGS && tar xf vdna_queryweb_v5.8.1.0_32bit.tar.gz && \
    cd vdna_queryweb_v5.8.1.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/queryweb/* $VDDB/queryweb/etc/

# query_runner
RUN cd $PKGS && tar xf vdna_query_runner_v5.8.1.0_64bit.tar.gz && \
    cd vdna_query_runner_v5.8.1.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/query_runner/* $VDDB/query_runner/etc/

# distridxd & utility
RUN cd $PKGS && tar xf vdna_distridx_v5.8.1.0_64bit.tar.gz && \
    cd vdna_distridx_v5.8.1.0/manager && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    sed -i "/ulimit/a export LC_ALL=C" $VDDB/distridxd/bin/server && \
    cp $ETC/distridxd/* $VDDB/distridxd/etc/ && \
    cd ../utility && \
    su vdna -c "mkdir -p $VDDB/utility; bash install.sh $VDDB/utility; exit 0" && \
    cp $ETC/utility/* $VDDB/utility/etc/

# qe
RUN cd $PKGS && tar xf vdna_qe_v5.9.2.0_32bit.tar.gz && \
    cd vdna_qe_v5.9.2.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/qe/* $VDDB/qe/etc/

# idistridxd && iutility
RUN cd $PKGS && tar xf vdna_imageindex_v5.9.0.0.tar.gz && \
    cd vdna_imageindex_v5.9.0.0/manager && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/idistridxd/* $VDDB/idistridxd/etc/ && \
    cd ../utility && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/iutility/* $VDDB/iutility/etc/

# image_qe
RUN cd $PKGS && tar xf vdna_image_qe_v5.9.2.0.tar.gz && \
    cd vdna_image_qe_v5.9.2.0 && \
    su vdna -c "bash Install.sh -d $VDDB -f $PKGS/global.json; exit 0" && \
    cp $ETC/image_qe/* $VDDB/image_qe/etc/

# VDNAGen
RUN cd $PKGS && dpkg -i VDNAGen_v3.5.4.3_linux_x86_64.deb

# vdna_query
RUN cd $PKGS && tar xf vdna_query_v5.10.0.0_linux_x86_32.tar.gz -C $VDDB && \
    echo "export PATH=$VDDB/vdna_query_v5.10.0.0_linux_x86_32/bin:$PATH" >> /home/vdna/.bashrc && \
    cp $ETC/vdna_query.py $VDDB/vdna_query_v5.10.0.0_linux_x86_32/bin

RUN cp $PKGS/restart.sh $VDDB/ && \
    cp $PKGS/stop.sh $VDDB/ && \
    chmod -R 777 $VDDB && \
    chown -R vdna:vdna $VDDB && \
    rm -rf $PKGS /var/lib/apt/lists/*

USER vdna
WORKDIR $VDDB

VOLUME ["$VOBILEDATA", "/var/lib/mysql"]

EXPOSE 80 443 4730 3306

ENTRYPOINT ["entrypoint.sh"]

CMD ["vddb"]
