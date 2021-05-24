FROM centos:latest

RUN yum install -y wget tar openssh-server openssh-clients sysstat sudo which openssl hostname
RUN yum install -y java-1.8.0-openjdk-devel
RUN yum install -y epel-release &&\
    yum install -y jq &&\
    yum install -y python3 &&\
    yum install -y nmap-ncat git
ARG MAVEN_VERSION=3.5.4
# Maven
RUN curl -fsSL https://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | tar xzf - -C /usr/share \
  && mv /usr/share/apache-maven-$MAVEN_VERSION /usr/share/maven \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_VERSION=${MAVEN_VERSION}
ENV M2_HOME /usr/share/maven
ENV maven.home $M2_HOME
ENV M2 $M2_HOME/bin
ENV PATH $M2:$PATH


RUN wget https://bootstrap.pypa.io/get-pip.py
RUN python3 get-pip.py
RUN pip3 install awscli

ENV SCALA_VERSION 2.13
ENV KAFKA_VERSION 2.7.0

RUN yum -y update && yum -y install tar gzip wget

RUN curl http://mirrors.gigenet.com/apache/kafka/${KAFKA_VERSION}/kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz | tar -zx -C /opt


RUN git clone https://github.com/aws-samples/mirrormaker2-msk-migration.git
WORKDIR mirrormaker2-msk-migration/CustomMM2ReplicationPolicy
RUN mvn clean install

# # Initialize the Kafka cert trust store
RUN find /usr/lib/jvm/ -name "cacerts" -exec cp {} /tmp/kafka.client.truststore.jks \;


ADD connect-distributed.properties /opt/connect-distributed.properties


RUN mv target/CustomMM2ReplicationPolicy-1.0-SNAPSHOT.jar /opt/kafka_2.13-2.7.0/libs

# include all start scripts
ADD start-kafka-connect.sh /opt/start-kafka-connect.sh
RUN mkdir -p /opt/logs
RUN chmod 777 /opt/start-kafka-connect.sh

# cleanup
RUN yum clean all;

EXPOSE 8083
USER root
ENTRYPOINT /opt/start-kafka-connect.sh