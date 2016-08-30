FROM centos:7

ENV MAKEOPTS="-j32"

MAINTAINER Kyle Travis <kyle@kyletravis.com>

# install document with yum
RUN sed -i 's/tsflags=nodocs//g' /etc/yum.conf

# install all software we need
RUN yum update -y
RUN yum install -y epel-release && \
 yum makecache && \
 yum install -y man passwd sudo tar which git mlocate links make bzip2 net-tools R perl postgresql-contrib \
 autoconf automake libtool m4 gcc gcc-c++ gdb bison flex cmake gperf maven indent wget \
 libuuid-devel krb5-devel libgsasl-devel expat-devel libxml2-devel \
 perl-ExtUtils-Embed pam-devel python-devel libcurl-devel snappy-devel \
 thrift-devel libyaml-devel libevent-devel bzip2-devel openssl-devel \
 openldap-devel protobuf-devel readline-devel net-snmp-devel apr-devel \
 libesmtp-devel xerces-c-devel python-pip json-c-devel libhdfs3-devel \
 apache-ivy java-1.6.0-openjdk-devel java-1.7.0-openjdk-devel \
 openssh-clients openssh-server && \
 yum clean all

# install cmake 3
RUN wget https://cmake.org/files/v3.6/cmake-3.6.1.tar.gz && \
 tar -zxf cmake-3.6.1.tar.gz && \ 
 cd cmake-3.6.1 && \
 ./bootstrap && \
 make ${MAKEOPTS} && \
 make install

# install google test package
RUN git clone https://github.com/google/googletest.git && \
 cd googletest && \
 mkdir build && cd build && \
 cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr .. && make ${MAKEOPTS} all install

# setup ssh server and keys for root
RUN sshd-keygen && \
 ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa && \
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
 chmod 0600 ~/.ssh/authorized_keys

# setup JAVA_HOME for all users
RUN echo "#!/bin/sh" > /etc/profile.d/java.sh && \
 echo "export JAVA_HOME=/etc/alternatives/java_sdk" >> /etc/profile.d/java.sh && \
 chmod a+x /etc/profile.d/java.sh

# install libyarn
RUN mkdir -p /tmp/build && \
 cd /tmp/build && git clone https://github.com/apache/incubator-hawq.git && cd /tmp/build/incubator-hawq/depends/libyarn && \
 mkdir build && cd build && \
 ../bootstrap --prefix=/usr && make ${MAKEOPTS} && make install && ldconfig

# create user gpadmin since HAWQ cannot run under root
RUN groupadd -g 1000 gpadmin && \
 useradd -u 1000 -g 1000 gpadmin && \
 echo "gpadmin  ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers.d/gpadmin

# sudo should not require tty
RUN sed -i -e 's|Defaults    requiretty|#Defaults    requiretty|' /etc/sudoers

ENV BASEDIR /data
RUN mkdir -p /data && chmod 777 /data && chown -R gpadmin: /tmp/build

USER gpadmin
ENV JAVA_HOME="/etc/alternatives/java_sdk"
ENV LDFLAGS="${LDFLAGS} -L${JAVA_HOME}/jre/lib/amd64/server"

# setup ssh client keys for gpadmin
RUN ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa && \
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
 chmod 0600 ~/.ssh/authorized_keys

# build hawq
RUN cd /tmp/build/incubator-hawq && \
 ./configure --prefix=/data/hawq-devel --with-r --with-python --with-java --with-perl --with-pgcrypto --with-openssl --enable-orca && \
 make ${MAKEOPTS} && \
 make install && \
 sed 's|localhost|centos7-namenode|g' -i /data/hawq-devel/etc/hawq-site.xml && \
 echo 'centos7-datanode1' >  /data/hawq-devel/etc/slaves && \
 echo 'centos7-datanode2' >>  /data/hawq-devel/etc/slaves && \
 echo 'centos7-datanode3' >>  /data/hawq-devel/etc/slaves && \
 source /data/hawq-devel/greenplum_path.sh
 
WORKDIR /data
