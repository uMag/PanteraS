# PanteraS
# Platform as a Service based on Paas in a box project
#
FROM ubuntu:14.04.3

MAINTAINER Wojciech Sielski "wsielski@team.mobile.de"

RUN locale-gen en_US.UTF-8

ENV DEBIAN_FRONTEND noninteractive
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV TERM xterm
ENV HOME /root
ENV GOPATH ${HOME}/go

RUN apt-get update \
    && apt-get install -y \
      apt-transport-https \
      python-pip \
      wget \
      curl \
      unzip \
      dnsutils \
      vim \
      git \
      lolcat \
      toilet \
      iptables \
      libapparmor1 \
      haproxy

RUN pip install supervisor-stdout
RUN pip install https://github.com/Supervisor/supervisor/archive/3.2-branch.zip

# install Java 8
RUN    echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" > /etc/apt/sources.list.d/java-trusty.list \
    && gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv EEA14886 && gpg --export --armor EEA14886 | apt-key add - \
    && apt-get update \
    && echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections \
    && apt-get -y install oracle-java8-installer \
    && apt-get clean \
    && update-alternatives --display java \
    && echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /etc/environment

ENV DOCKER_APP_VERSION            1.8.3-0~trusty
ENV MESOS_APP_VERSION             0.25.0-0.2.70.ubuntu1404
ENV MARATHON_APP_VERSION          0.11.1-1.0.432.ubuntu1404
#ENV REGISTRATOR_APP_VERSION       v6

ENV DOCKER_HOST unix:///tmp/docker.sock

# DNSMASQ
#
RUN apt-get install -y dnsmasq dnsutils

# ZOOKEEPER MESOS MARATHON
#
RUN echo "deb http://repos.mesosphere.io/ubuntu/ trusty main" > /etc/apt/sources.list.d/mesosphere.list

#RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF && gpg --export --armor E56151BF | apt-key add -

RUN apt-get update \
    && apt-get -y install \
      mesos=${MESOS_APP_VERSION} \
      marathon=${MARATHON_APP_VERSION}

ADD zkStart.sh /opt/zkStart.sh
RUN rm /etc/mesos/zk

#RUN echo 'deb http://get.docker.io/ubuntu docker main' > /etc/apt/sources.list.d/docker.list
#RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D8576A8BA88D21E9
#RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv D8576A8BA88D21E9 && gpg --export --armor D8576A8BA88D21E9 | apt-key add -

RUN echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv F76221572C52609D && gpg --export --armor F76221572C52609D | apt-key add -
RUN apt-get update \
    && apt-get install -y docker-engine=${DOCKER_APP_VERSION} \
    || { wget http://apt.dockerproject.org/repo/pool/main/d/docker-engine/docker-engine_${DOCKER_APP_VERSION}_amd64.deb \
    && dpkg -i docker-engine_${DOCKER_APP_VERSION}_amd64.deb; }

#Adding bamboo
RUN mkdir -p /run/haproxy && \
    mkdir -p /var/log/supervisor $$ \
    mkdor -p /opt/bamboo/config

ADD ./config /opt/bamboo/config
ADD bamboo /opt/bamboo/

VOLUME /var/log/supervisor

# Font for logo
RUN wget https://raw.githubusercontent.com/patorjk/figlet.js/master/fonts/Elite.flf -O /usr/share/figlet/Elite.flf

ADD supervisord.conf   /etc/supervisord.conf
ADD supervisord.sh     /opt/
ADD version            /opt/

WORKDIR /opt
ENTRYPOINT [ "/opt/supervisord.sh" ]
EXPOSE 80 8000