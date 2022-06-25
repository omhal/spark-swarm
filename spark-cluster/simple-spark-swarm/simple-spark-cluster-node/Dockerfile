FROM debian

#
# Base image for Apace Spak standalone cluster. 
#
# Inspired by https://hub.docker.com/r/gettyimages/spark/dockerfile
#
#
# Expected volumes:
#	/app/spark - this is the spark working directory
#	<data dir> - All nodes should mount the same data directory. This can be a GlusterFS or NFS mount.
#
# Expected service names:
#	spark-master - the service where the spark master runs
#

ARG SPARK_VERSION=3.3.0
ARG HADOOP_MINOR_VERSION=3.3
ARG HADOOP_VERSION=3.3
ARG SCALA_VERSION=2.13.1
ARG SPARK_PACKAGE=3.3.0

RUN apt-get update \
 && apt-get install -y locales \
 && dpkg-reconfigure -f noninteractive locales \
 && locale-gen C.UTF-8 \
 && /usr/sbin/update-locale LANG=C.UTF-8 \
 && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
 && locale-gen \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
 
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN apt-get update \
 && apt-get install -y curl unzip procps \
    python3 python3-setuptools \
 && ln -s /usr/bin/python3 /usr/bin/python \
 && apt-get install -y python3-pip \
 && pip3 install py4j \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

ENV PYTHONIOENCODING UTF-8
ENV PIP_DISABLE_PIP_VERSION_CHECK 1

# JAVA & SCALA
RUN apt-get update \
 && apt-get install -y openjdk-11-jre \
 && apt-get remove scala-library scala  \
 && curl -o scala-${SCALA_VERSION}.deb https://www.scala-lang.org/files/archive/scala-${SCALA_VERSION}.deb \
 && dpkg -i scala-${SCALA_VERSION}.deb \
 && apt-get clean \
 && rm scala-${SCALA_VERSION}.deb \
 && rm -rf /var/lib/apt/lists/*

# create the user software will run from
RUN useradd -m -s /bin/bash spark


# SPARK
ENV SPARK_PACKAGE spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION}
ENV SPARK_HOME /usr/spark-${SPARK_VERSION}
#ENV SPARK_DIST_CLASSPATH=""
ENV HADOOP_CONF_DIR=${SPARK_HOME}/conf/
ENV PATH $PATH:${SPARK_HOME}/bin
COPY spark-3.3.0.tgz /home
RUN tar xzf /home/spark-3.3.0.tgz -C /usr/ \
        && rm -rf /home/spark-3.3.0.tgz \
        && cp -al /usr/spark-3.3.0/* $SPARK_HOME \
	&& chown -R root:root $SPARK_HOME \
	&& ln -s $SPARK_HOME /usr/local/spark
RUN mkdir -p /app/spark \
 && chown spark -R /app/spark

# add python libraries useful in PySpark
RUN python3 -mpip install matplotlib \
	&& pip3 install pandas seaborn

# copy Spark configurations
COPY ./spark-conf/* $SPARK_HOME/conf/

# set up command
COPY start-spark-node.sh /
COPY start-spark-master.sh /
USER spark
WORKDIR /home/spark
CMD ["/bin/bash", "/start-spark-node.sh"]
