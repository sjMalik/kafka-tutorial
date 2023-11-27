
## Single Node Kafka Broker
![](./single_node_kafka_broker.webp)

## What is Kafka?

**Definition:**
The Kafka is an events streaming platform. Events, or messages, represent the actual data that is exchanged through Kafka. It is arguably the most popular messaging platform in the world. In Kafka's world, there are data publishers called producers who push messages into Kafka. And there are subscribers called consumers which listen and receive the messages.

**Capabilities Kafka provides for Data Exchange:**
- It collects messages from multiple producers concurrently. 
- It provides persistent storage of the messages received. 
- This provides fault-tolerance capabilities. 
- It transports data across from the producers to the consumers. 
- With mirroring capabilities, it can also transport across networks. 
- It distributes data to multiple concurrent consumers for downstream processing. 
- Finally, it provides tracking of message consumption by each consumer. This ensures at least one's delivery of the messages, even if the consumers go down and come back again. 

## What is Broker?

A Kafka broker is a single Kafka server or instance running in a Kafka cluster. Brokers are responsible for handling and managing the storage, processing, and distribution of data (messages) within Kafka topics.

- The Kafka broker receives messages from producers and stores them locally in logs. 
- Consumers subscribe to specific topics within the Kafka broker. 
- The broker keeps track of all the active consumers. 
- It knows about the last message that was sent to the consumer, so it only sends new messages in the subscribed topics to that consumer. 
- It also keeps a heartbeat with every consumer so when a consumer dies, it can track and reset. 
Kafka brokers manage the lifecycle of topics. 

## What is the Topic?
A topic in Kafka is an entity that holds messages. It's similar to a file that contains papers where the messages represent the papers. It's similar to a database table that contains records where the messages represent the records. Topics can be considered as a queue for similar messages. 

- Each topic supports multiple producers to publish data to the topic concurrently
- Similarly, multiple consumers can consume data from the topic.

## Kafka Producer:
A Kafka producer is responsible for publishing data to Kafka topics. It sends messages to specific topics within the Kafka cluster. Producers write data to topics that are then made available to consumers. Producers can send messages synchronously or asynchronously, and they specify the topic to which they are producing messages.

## Kafka Consumer:
A Kafka consumer reads messages from Kafka topics. Consumers subscribe to specific topics to receive messages published to those topics by producers. They can read messages at their own pace, and they can be part of a consumer group where multiple consumers collectively consume messages from a topic, ensuring load balancing and fault tolerance.

We are going to set up a Kafka broker in Docker. If you're using either Linux or Windows, please install Docker Desktop from the Docker website. You can check my video to install docker on windows.
1. We will set up a single Kafka container. 
2. And Set up Kafdrop, a web-based UI tool for monitoring Apache Kafka clusters

The Docker Compose configuration for the same is available in the kafka-single-node.yml file. 
- We have the Kafka service, which is from Bitnami. We will be using Kafka version 3.4. 
- We will be exposing two ports in Kafka. One port, 29092, will be used for other Docker containers to communicate with this container. This port should only be used when the client is another Docker container. 
A second port, 9092, will be used when accessing Kafka from the host system. 
-  We are enabling KRaft, which is the newer configuration management option in Kafka using the parameter
```js
KAFKA_ENABLE_KRAFT=yes. 
```
- We set to allow plain text listeners in this installation. This is recommended only for development setup. - The NODE_ID for this broker is set to 1. This should be unique within the cluster. 
- Each Kafka node can perform two roles, namely broker and controller. Being a single node system, we will choose both. 
- The CONTROLLER_LISTENER_NAME is set as CONTROLLER. It'll be referred to in other configuration parameters. - Similarly, the INTER_BROKER_LISTENER_NAME is set as CLIENT. 
- Listener configuration lists the ports on which Kafka brokers will listen. The client listener shows the port on which this broker listens for internal communications within the Docker environment. The external listener listens for communications outside the Docker environment. The controller listens on port 9093. 
- The security protocol is set as plain text for all the listeners. This is recommended only for learning and development environments. The advertised listeners are those that allow clients to connect Kafka. We only expose the client and external but not the controller. 
- The CONTROLLER_QUORUM_VOTERS identifies all the brokers that perform the role of a controller in a cluster. As the cluster has only one node that behaves as a controller, we list its ID, IP address, and port. 
- Finally, the port 9092 is exposed outside the Docker environment for external clients to connect. 

Let's start the container now. Open the terminal window. navigate to the directory where kafka-single-node.yml file exists. Now run the docker-compose command. 
The command would be: 
```js
docker-compose -f kafka-single-node.yml up -d
```
This will download the images if they are not already on your local Docker and then proceed to start the containers. It will take some time.
Then check if the container is up and running with the command: 
```js
docker ps
```
2. Represent Kafdrop UI

## Video 2:
1. We'll create the two topic `Topic1` and `Topic2`. First one we create client shell scripts that are available in kafka. We will explore the command line options in Kafka for managing topics as well as publishing and subscribing. In this video, we will explore various client shell scripts that are available in kafka. To begin, we need to log into the kafka container using the docker exec command. We do so by calling 
```js
docker exec -it kafka-broker /bin/bash // Logging into the Kafka Container
```
This takes us inside the docker container. 
Now, let's navigate to the bin directory for kafka here. 
```js
cd /opt/bitnami/kafka/bin   //Navigate to the Kafka Scripts directory
```
The bin directory contains a number of shell scripts for kafka management, publishing, and subscribing. The bin directory contains a number of shell scripts that can be used to interact with kafka. 

2. ## Creating new Topics
```js
        ./kafka-topics.sh \
            --bootstrap-server localhost:29092 \
            --create \
            --topic Topic1 \
            --partitions 1 \
            --replication-factor 1
```
First, we need to provide a link to the Kafka broker. This is provided with the parameter bootstrap-server. Note that we are using the internal port 29092 as we are accessing the broker from inside the docker cluster. 
Then We provide the name of the topic in the topic parameter. While this name can be any string, 
We then specify two mandatory parameters, the number of partitions and the replication factor. 
We keep the partition size to one and replication to one since we only have one Kafka broker running. We will discuss partitions and replication in later videos.

3. Next topics we'll create from kafdrop
4. Getting details about a Topic
```js
        ./kafka-topics.sh \
            --bootstrap-server localhost:29092 \
            --describe
```
When you execute this command, it communicates with the Kafka broker running on localhost at port 29092. The purpose is to gather information about all the available topics within the Kafka cluster connected to this broker.

Upon execution, this command retrieves and displays details about existing Kafka topics. It showing information such as the topic name, number of partitions, replication factor, and configuration settings associated with each topic. This command provides a high-level summary of the Kafka topics present in the cluster connected to the specified broker.

5. Publishing Messages to Topic1
```js
        ./kafka-console-producer.sh \
            --bootstrap-server localhost:29092 \
            --topic Topic1
```
This command, when executed, starts a console-based producer that connects to a Kafka broker running locally on port 29092. Once the producer is running, it's configured to send messages to the Kafka topic named Topic1. Any text entered in the console after executing this command will be treated as a message and will be sent to the specified Kafka topic (Topic1). This allows users to manually input messages that will be published to the specified Kafka topic in real-time via the command line.

6. Consuming Messages from Topic1: Consumer1
```js
        ./kafka-console-consumer.sh \
            --bootstrap-server localhost:29092 \
            --topic Topic1
```
This command, when executed, starts a console-based consumer that connects to a Kafka broker running locally on port 29092. Once the consumer is running, it's configured to read and display messages from the Kafka topic named Topic1.

Any messages that are produced and sent to the Topic1 Kafka topic will be consumed and displayed in the console where this command is executed. This allows users to see messages being published to the specified Kafka topic (Topic1) in real-time via the command line.

7. Consuming Messages from Topic1: Consumer2
```js
        ./kafka-console-consumer.sh \
            --bootstrap-server localhost:29092 \
            --topic Topic1 \
            --from-beginning
```
However, the significant difference here is the --from-beginning flag. By including --from-beginning, the consumer will start consuming messages from the very beginning of the topic, meaning it will read all the messages available in the topic, starting from the earliest available offset.

This is particularly useful when you want to read all the historical messages in a topic rather than only receiving new messages that are produced after the consumer has started. The consumer will display all past and future messages from the specified Kafka topic on the command line.

8. Publishing Messages to Topic2
```js
        ./kafka-console-producer.sh \
            --bootstrap-server localhost:29092 \
            --topic Topic2
```

9. Consuming Messages from Topic2: Consumer3
```js
        ./kafka-console-consumer.sh \
            --bootstrap-server localhost:29092 \
            --topic Topic2
```
