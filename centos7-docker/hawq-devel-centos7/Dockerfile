FROM centos:7

ENV MAKEOPTS="-j32"
ENV GIT="/usr/local/bin/git clone -j8"

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
 libesmtp-devel libxerces-c-dev xerces-c-devel python-pip json-c-devel libhdfs3-devel \
 apache-ivy java-1.6.0-openjdk-devel java-1.7.0-openjdk-devel gettext \
 openssh-clients openssh-server && \
 yum clean all

# install latest version of git
RUN git clone https://github.com/git/git.git && \
 cd git && \
 git checkout v2.9.3 && \
 make configure && \
 ./configure --prefix=/usr/local && \
 make ${MAKEOPTS} install

# install cmake 3
RUN wget https://cmake.org/files/v3.6/cmake-3.6.1.tar.gz && \
 tar -zxf cmake-3.6.1.tar.gz && \ 
 cd cmake-3.6.1 && \
 ./bootstrap && \
 make ${MAKEOPTS} && \
 make install

# install google test package
RUN ${GIT} https://github.com/google/googletest.git && \
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
RUN mkdir -p /tmp/build && cd /tmp/build && \
 ${GIT} https://github.com/apache/incubator-hawq.git && \
 cd /tmp/build/incubator-hawq/depends/libyarn && \
 mkdir build && cd build && \
 ../bootstrap --prefix=/usr && make ${MAKEOPTS} && make install && ldconfig && \
 rm -rf /tmp/build

# create user gpadmin since HAWQ cannot run under root
RUN groupadd -g 1000 gpadmin && \
 useradd -u 1000 -g 1000 gpadmin && \
 echo "gpadmin  ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers.d/gpadmin

# sudo should not require tty
RUN sed -i -e 's|Defaults    requiretty|#Defaults    requiretty|' /etc/sudoers

ENV BASEDIR /data
RUN mkdir -p /data && chmod 777 /data

USER gpadmin
ENV JAVA_HOME="/etc/alternatives/java_sdk"
ENV LDFLAGS="${LDFLAGS} -L${JAVA_HOME}/jre/lib/amd64/server"

# setup ssh client keys for gpadmin
RUN ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa && \
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
 chmod 0600 ~/.ssh/authorized_keys

# build hawq
RUN ${GIT} --recursive https://github.com/apache/incubator-hawq.git /tmp/incubator-hawq && \
 cd /tmp/incubator-hawq && \
 ./configure --prefix=/data/hawq-devel --with-r --with-python --with-java --with-perl --with-pgcrypto --with-openssl --enable-orca && \
 make ${MAKEOPTS} && \
 make install && \
 rm -rf /tmp/incubator-hawq && \
 sed 's|localhost|centos7-namenode|g' -i /data/hawq-devel/etc/hawq-site.xml && \
 echo 'centos7-datanode1' >  /data/hawq-devel/etc/slaves && \
 echo 'centos7-datanode2' >>  /data/hawq-devel/etc/slaves && \
 echo 'centos7-datanode3' >>  /data/hawq-devel/etc/slaves && \
 source /data/hawq-devel/greenplum_path.sh
 
WORKDIR /data
