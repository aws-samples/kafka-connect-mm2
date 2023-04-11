## Run Kafka Connect as Fargate Docker containers and deploy MirrorMaker configuration files

This repository includes dockerfile, fargate task definition and mirror maker configuration files.

### Kafka Connect

[**Kafka Connect**](https://kafka.apache.org/documentation/#connect), an open source component of Apache Kafka is a tool for scalably and reliably streaming data between Kafka and other external systems. It makes it simple to quickly define connectors that move large collections of data into and out of Kafka. Kafka connect can operate on a Amazon Managed Streaming for Apache Kafka (Amazon MSK) cluster or on a self managed Apache Kafka cluster running on AWS. 

#### Clone the repository and build the container.  

    cd connect-distributed-docker
    docker build .

Tag the docker image and push it to a container registry before deploying it as a fargate task.
connect-distributed.properties file in the docker is configured to connect to a Kafka cluster configured with SASL/SCRAM authentication. 

To deploy fargate task, make sure to replace following variables in task.json found under **fargate** folder.  

* **IMAGE_URL** - container registry url of the docker image
* **IAM_ROLE** - ECS Task role with permission to interact with MSK clusters, read secrets from secret manager, decrypt KMS key used to encrypt the secret, read images from ECR, and write logs to cloudwatch.  

Environment Variables - 

*	**BROKERS** –  Bootstrap servers connection string of Amazon MSK cluster / Apache Kafka cluster on which Kafka connect is to be deployed
*	**USERNAME** – Username configured as part of SASL/SCRAM configuration of Kafka. In Amazon MSK configuration, this will be the secret in AWS secrets manager associated with the cluser.
*	**PASSWORD** – Password configured as part of SASL/SCRAM configuration of Kafka. In Amazon MSK configuration, this will be the secret in AWS secrets manager associated with the cluser.
*	**GROUP** – Group name. Can be any random string that will be used as suffix when kafka connect topics are created for Kafka connect.

### Mirror Maker

**Mirror Maker** is a tool built to replicate data between two Kafka environments in streaming manner. Mirrormaker 2.0 is built on top of Kafka connect framework and is used to replicate topics, topic configurations, consumer groups and their offsets, and ACLs from one or more source Kafka clusters to one or more target Kafka clusters, i.e., across cluster environments. In a nutshell, MirrorMaker uses Connectors to consume from source clusters and produce to target clusters.

To run a successful MirrorMaker 2.0 deployment, you need 3 different connectors 

* **MirrorSourceConnector** - This connector is used to replicate topics and metadata from one Kafka cluster to another. This connector will read from an external Kafka cluster and write it to the Kafka cluster on which the Kafka connect is running.
* **HeartBeatConnector** - This connector emits a heartbeat topic which gets replicated to demonstrate connectivity across clusters. This is to ensure the connectivity between two clusters are active.
* **CheckpointConnector** - This connector is responsible to emit checkpoints in the secondary cluster containing offsets for each consumer group in the primary cluster. This is responsible for replicating consumer offset from one cluster to another.

Sample connector configuration file for each of these connector are available under **connect-config** folder. Make sure to replace **PRIMARY_BOOTSTRAP_URL**, **SECONDARY_BOOTRSTRAP_URL**, **USERNAME** and **PASSWORD** variables in the configuration files (to applicable values).

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

