# Use maven to compile the java application.
FROM maven:3-jdk-11-slim AS build-env

RUN cd / && curl -sL -o - https://github.com/aws-samples/mirrormaker2-msk-migration/archive/refs/heads/master.zip | jar xv

# Set the working directory to /app
WORKDIR /mirrormaker2-msk-migration-master/CustomMM2ReplicationPolicy

# Compile the application.
RUN mvn clean install

# Build runtime image. 6.2.2 is for kafka2.8
FROM confluentinc/cp-kafka:6.2.2 as mm2 

# Copy the compiled files over.
COPY --from=build-env /mirrormaker2-msk-migration-master/CustomMM2ReplicationPolicy/target/CustomMM2ReplicationPolicy-1.0-SNAPSHOT.jar /usr/share/java/kafka/

COPY connect-distributed.properties /opt/connect-distributed.properties
COPY start-kafka-connect.sh /opt/start-kafka-connect.sh

USER root
RUN mkdir -p /opt/logs && \
  chmod +x /opt/start-kafka-connect.sh

USER appuser

EXPOSE 8083

ENTRYPOINT /opt/start-kafka-connect.sh
