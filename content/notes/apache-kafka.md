+++
title = "Apache Kafka (WIP)"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

{{% notice warning %}}
2022-07-11: This is a WORK IN PROGRESS document (WIP) and need to be reviewed.
{{% /notice %}}

{{< toc >}}

## Apache Kafka

Why Apache Kafka?

- Created by LinkedIn, now Open Source Project Maintained by Confluent (Apache Stewardship)
- Distributedm resilient architecture, fault tolerant
- scales Horizontaly (100s of borkers, millions of messages per seconds, etc.)

Use Cases?

- Messaging system
- Activity tracking
- Gather metrics from many different locations
- Application logs gathering
- Stream processing (e.g., kafka stream API, Apache Spark, etc.)
- De-coupliung system dependicies
- Integration with Spark, Flink, Storm, Hadoop, and other big Data technologies

### Topics, partitions and offsets

- `Topics`: a particular stream of data
  - similar to a table in a DB (but without the constraints)
  - you can have as many topics as you want
  - a topic is identified by its name
- `Partitions`: spliting topics
  - each partiction is orderied
  - each message within a partiction gets an incremental id, called `offset`

  ```text
              | Partition 0 -> offset: | 0 | 1 | 2 | 3 | 4 | 5 | ...
              |
  Kafka Topic + Partition 1 -> offset: | 0 | 1 | 2 | ... writes
              |
              | Partition 2 -> offset: | 0 | 1 | 2 | 3 | 4 | ...
  ```

  - offset only have a meaning for a specific partiction (e.g., ofsset 3 in partition 0 != offset 3 in partition 1)
  - order is guaranteed only within a partition (not acresoo partitions)
  - data is kept only for a limited time (default is one week)
  - once the data is written to a partition, it can't be changed (immutability)
  - data is assigned randomly to a partition unless a key is provided (more on this later)

### Brokers

- a `kafka cluster` in composed of multiple brokers (broker = servers)
- each broker is identified with its ID (integer)
- each broker contains certain topic partitions
- after connecting to any broker (called a bootstrap broker), you will be connected to the intire cluster
- a good number to get started is `3 brokers`, but may go over 100 brokers

### Brokers and topics

- Example of Topic-A with 3 partitions
- Exmaple of Topic-B with 2 partitions

| Broker 101 | Broker 102 | Broker 103 |
|:---:|:---:|:---:|
| Topic A partition 0 | Topic A partition 2 | Topic A partition 1 |
| Topic A partition 1 | Topic A partition 0 | |

### Topic replication factor

- topics should have a replication factor > 1 (usually between 2 and 3)
- this way if a broker is down, another broker can serve the data
- example: topic-A with 2 partitions and replications factor of 2

| Broker 101 | Broker 102 | Broker 103 |
|:---:|:---:|:---:|
| Topic-A Partition 0 | Topic-A Partition 1 | Topic-A Partition 1 |
| | Topic-A Partition 0 | |

- example: broker 102 goes down; broker 101 and 103 still up, both partitions still work
- at any time only `one broker` can be a leader for a given partition
- only that leader can receive and serve data for a partition
- the other brokers will synchronize the data
- therefore each partion has one leader and multiple ISR (in-sync replica)

### Producers

- producers write data to topics (which is made of partitions)
- producers automatically know to which broker and partition to write to
- in case of broker failures, producers will automatically recover
- producers can choose to receive acknowledgement of data writes
  - acks=0: producers won't wait for acknowledgment (possible data loss)
  - (default) acks=1: producer will wait for leader acknowledgment (limited data loss)
  - acks=all: leaders and replicas acknowledgment (no data loss)

#### Message keys

- producers can choose to send a `key` with the message (string, number, etc.)
- if key=null, data is sent round robin (broker 101, 102 then 103, and 101 again...)
- if a key is sent, then all messages for that key will always goto the same partition
- a key is basically sent if you need a message ordering for specific field (e.g., truck_id, etc.)
  - we get this guarantee due to key hashing, which depends on the number of partitions

### Consumers

- consumers read data from a tipic (identified by name)
- consumers know which broker to read from
- in case of broker failures, consumer know how to recover
- data is read in order `within each partitions`

#### Consumers groups

- consumers read data in consumer groups
- each consumer within a group reads from exclusinve partitions
- if youy have more consumer than partitions, some consumers with be icative
- __if you have more consumers than partitions, some consumers will be inactive__

### Consumer offsets

- `kafka` stores the offsets at which a consumer group has beed reading
- the offset committed live in a kafka `topic` named `__consumer_offsets`
- when a consumer in a group has processed data received from kafka, it should be committing the offsets
- if a consumer dies, it will be able to read back from where it left off (due to the committed consumer offsets)

#### Delivery semantics for consumers

- consumers choose when to commit offsets
- there are 3 delivery semantics
- at most once:
  - offsets are committed as soon as the message is received
    - offsets are commited as soon as the message is received
    - if the processing goes wring, the message will be lost (it won't be read again)
  - at least once (usually preferred)
    - offsets are committed after the message is processed
    - if the processing goes wring, the message will be read again
    - this can reults in duplicate processing of messages, so make sure the processing is `idempotent`
  - exactly once
    - can be achieved for `kafta-to-kafka` workflows using kafka streams API
    - for `kafka-to-external-system workflows`, it requires the consumer to be `idempotent`

### Kafka broker discovery

- every kafka broker is also called a `bootstrap server`
- that means that `you only need to connect to one broker` and you will be connected to the entire cluster
- each broker knows about all brokers, topics and partitions (metadata)

### Zookeeper

- zookeeper manages brokers (keeps a list of them)
- zookeeper helps in performing leader election partitions
- zookeeper sends notifications to kafka in case of changes (e.g., new topic, broker dies, brker comes up, delete topics, etc.)
- _kafka can't work without zookeeper_
- zookeeper _by design_ operates with an odd number of servers (3, 5, 7, ...)
- zookeeper has a leader (handle writes) and the rest if the servers are followers (handle reads)
- zookeeper `does not` store consumer offsets with kafka > v0.10

### Kafka guarantees

- Messages are appended to a `topic-partition` in othe order they are sent
- consumers read messages in the order stored in a `topic-partition`
- with a replication factor of N, producers and consumers can tolerate up to `N-1` brokers being down
- this is why a replicator factor of 4 is a good idea:
  - allows for one broker to be taken down for maintenance
  - allows for another broker to be taken down unexpectedly
- as long as the number of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition (i.e., hashed key)

### Install Kafka using Docker images

Create the `docker-compose.yaml` file:

```yaml
version: '3.5'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

From the CLI:

```bash
> docker-compose up -d
> docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS          PORTS                                         NAMES
ab09b5c8bc7b   confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   11 minutes ago   Up 11 minutes   9092/tcp, 0.0.0.0:29092->29092/tcp            kafka-cluster_kafka_1
8e7725b7874b   confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   11 minutes ago   Up 11 minutes   2888/tcp, 3888/tcp, 0.0.0.0:22181->2181/tcp   kafka-cluster_zookeeper_1

> $ netstat -aon | grep 22181
  TCP    0.0.0.0:22181          0.0.0.0:0              LISTENING       23792
  TCP    [::]:22181             [::]:0                 LISTENING       23792
  TCP    [::1]:22181            [::]:0                 LISTENING       27612

> netstat -aon | grep 29092
  TCP    0.0.0.0:29092          0.0.0.0:0              LISTENING       23792
  TCP    [::]:29092             [::]:0                 LISTENING       23792
  TCP    [::1]:29092            [::]:0                 LISTENING       27612

> docker-compose logs kafka | grep -i started
kafka_1      | [2022-05-31 14:49:34,438] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2022-05-31 14:49:34,441] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
kafka_1      | [2022-05-31 14:49:34,445] INFO [SocketServer listenerType=ZK_BROKER, nodeId=1] Started data-plane acceptor and processor(s) for endpoint : ListenerName(PLAINTEXT) (kafka.network.SocketServer)
kafka_1      | [2022-05-31 14:49:34,450] INFO [SocketServer listenerType=ZK_BROKER, nodeId=1] Started data-plane acceptor and processor(s) for endpoint : ListenerName(PLAINTEXT_HOST) (kafka.network.SocketServer)
kafka_1      | [2022-05-31 14:49:34,450] INFO [SocketServer listenerType=ZK_BROKER, nodeId=1] Started socket server acceptors and processors (kafka.network.SocketServer)
kafka_1      | [2022-05-31 14:49:34,459] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
kafka_1      | [2022-05-31 18:51:50,791] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2022-05-31 18:51:50,791] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
kafka_1      | [2022-05-31 19:52:52,922] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2022-05-31 19:52:52,923] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
kafka_1      | [2022-05-31 21:02:51,738] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2022-05-31 21:02:51,739] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
kafka_1      | [2022-05-31 23:24:34,226] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2022-05-31 23:24:34,226] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
```

Note: `nc -z localhost <port>` can be used on MacOS and linux.

Follow the [Guide to Setting Up Apache Kafka Using Docker](https://www.baeldung.com/ops/kafka-docker-setup) instructions.

Install the [Kafka Offset Explorer](https://kafkatool.com/download.html) to connect to Kafka Cluster and make sure to configure the bootstrap server: `localhost:29092`

### Use Kafka Topics CLI

Use `docker` to execute the `kafka-topics` command:

```bash
> docker-compose ps
          Name                       Command            State                      Ports
-----------------------------------------------------------------------------------------------------------
kafka-cluster_kafka_1       /etc/confluent/docker/run   Up      0.0.0.0:29092->29092/tcp, 9092/tcp
kafka-cluster_zookeeper_1   /etc/confluent/docker/run   Up      0.0.0.0:22181->2181/tcp, 2888/tcp, 3888/tcp

> docker exec -t kafka-cluster_kafka_1 kafka-topics
Create, delete, describe, or change a topic.
Option                                   Description
------                                   -----------
--alter                                  Alter the number of partitions,
                                           replica assignment, and/or
                                           configuration for the topic.
.
.
.
```

To use Kafka Container shell:

```bash
> docker exec -it kafka-cluster_kafka_1 sh
sh-4.4$
```

### Topics CLI

Reference: [Apache Kafka CLI commands cheat sheet](https://medium.com/@TimvanBaarsen/apache-kafka-cli-commands-cheat-sheet-a6f06eac01b#8c2f)

#### Creating a topic

```bash
sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --create --topic first_topic --partitions 3 --replication-factor 1

WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic first_topic.

sh-4.4$
```

Note: the `--bootstrap-server localhost:9092` replaces the `kafka-topics` command `--zookeeper localhost:9092`

#### Listing current topics

```bash
sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --list

first_topic

sh-4.4$
```

#### Describe a topic

```bash
sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --describe --topic first_topic

Topic: first_topic      TopicId: vQ6o8fX1Sx-qNQuUQ5vbAg PartitionCount: 3       ReplicationFactor: 1    Configs:
        Topic: first_topic      Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: first_topic      Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: first_topic      Partition: 2    Leader: 1       Replicas: 1     Isr: 1

sh-4.4$
```

#### Delete a topic

```bash
sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --list

first-topic
first_topic

sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --delete --topic first-topic

sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --list

first_topic

sh-4.4$
```

### Kafka console producer

#### Create messages

Creating 4 messages using Kafka Broker using the `kafka-console-producer` command.  The `CTRL-C` command will make the `kafka-console-producer` command to stop.

```bash
sh-4.4$ kafka-console-producer --broker-list localhost:9092 --topic first_topic

>message1
>message2
>message3
>message4
>^C

sh-4.4$
```

#### Create messages in a non-existing topic

It is possible to create a new topic on-the-fly when adding but it is not recommended since the default values for both `PartitionCount` and `ReplicationFactor` are set to `1`. Best practices require more partitions and replications.

Default replication value can be changed from `/etc/kafka/server.properties` value `num.partitions`, default is 1.

```bash
sh-4.4$ kafka-console-producer --broker-list localhost:9092 --topic new_topic

>This is a message to a new topic
[2022-06-02 18:56:53,185] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 3 : {new_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
>another message

sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --list

first_topic
new_topic

sh-4.4$ kafka-topics --bootstrap-server localhost:9092 --topic new_topic --describe

Topic: new_topic        TopicId: FuC1JLRETNCrgUWaEPZCOg PartitionCount: 1       ReplicationFactor: 1    Configs:
        Topic: new_topic        Partition: 0    Leader: 1       Replicas: 1     Isr: 1

sh-4.4$
```

#### Change the producer-property

Changing the `acks` property.  Refer to [producer](#producers) above sections for more information about `acks=all` property.

```bash
sh-4.4$ kafka-console-producer --broker-list localhost:9092 --topic first_topic --producer-property acks=all

>acks message1
>^C

sh-4.4$
```

### Kafka console consumer

#### Consuming messages from a topic

The `kafka-console-consumer` won't consume topic messages from `offset=0` by default to avoid consuming millions of existing message.  It will consume the upcomming messages only (from now on).  To get all messages from offest:0, the `--from-beginning` option must be specified.

```bash
sh-4.4$ kafka-console-consumer --bootstrap-server localhost:9092 --topic first_topic --from-beginning

message2
acks message1
message3
message1
message4
```

The order isn't garanteed when the number of topic partitions is greater than 1, as explained in [Topics, partitions and offsets](#topics-partitions-and-offsets) section.

#### Consuming messages from a topic with groups

The `--group` define a group of kafka consumers that will share the consumption load for one given topic.

The `kafka-console-producer` example:

```bash
sh-4.4$ kafka-console-producer --broker-list localhost:9092 --topic first_topic

>patate
>carotte
>pomme
>orange
>
```

The first `kafka-console-consumer` example:

```bash
sh-4.4$ kafka-console-consumer --bootstrap-server localhost:9092 --topic first_topic --group my-group

carotte
orange
```

The second `kafka-console-consumer` example:

```bash
sh-4.4$ kafka-console-consumer --bootstrap-server localhost:9092 --topic first_topic --group my-group

patate
pomme
```

#### Consumer groups command

Get all the available consumer groups:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --list

my-group

sh-4.4$
```

Get consumer group information:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                           HOST            CLIENT-ID
my-group        first_topic     0          3               3               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer
my-group        first_topic     1          2               2               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer
my-group        first_topic     2          4               4               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer

sh-4.4$
```

#### Consumer groups: reseting offsets

All messages are currently consumed: `current-offsets=log-end-offsets`:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                           HOST            CLIENT-ID
my-group        first_topic     0          3               3               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer
my-group        first_topic     1          2               2               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer
my-group        first_topic     2          4               4               0               console-consumer-13be95b9-8704-4471-8211-2660c5ab59b1 /172.19.0.3     console-consumer

sh-4.4$
```

Reseting the `current-offsets` to `0` using `--to-earliest` option:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --group my-group --reset-offsets --to-earliest --execute --topic first_topic

GROUP                          TOPIC                          PARTITION  NEW-OFFSET
my-group                       first_topic                    0          0
my-group                       first_topic                    1          0
my-group                       first_topic                    2          0

sh-4.4$
```

Verify the group offsets is not `0`:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group

Consumer group 'my-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my-group        first_topic     0          0               3               3               -               -               -
my-group        first_topic     1          0               2               2               -               -               -
my-group        first_topic     2          0               4               4               -               -               -

sh-4.4$
```

Shift the offset by `1` using `--shift-by` option:

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --group my-group --reset-offsets --shift-by 1 --
execute --topic first_topic

GROUP                          TOPIC                          PARTITION  NEW-OFFSET
my-group                       first_topic                    0          1
my-group                       first_topic                    1          1
my-group                       first_topic                    2          1

sh-4.4$
```

Available `--reset-offsets` options are :

```bash
--to-datetime
--by-period
--to-earliest
--to-latest
--shift-by
--from-file
--to-current
```

### Kafka-Client with Java

Use [Kafka Documentation](https://kafka.apache.org/documentation/) as much as possible.

#### Create Java project

Using IntelliJ:

1. create a project
1. search for `kafka-client` in [https://mvnrepository.com/](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)
1. select the desired version (i.e., the following will use 3.2.0)
1. copy the `Gradle` implementation information in the `dependencies` section of the `build.gradle` file
1. download the dependencies using IntelliJ `Gradle` window -> `Reload...` button
1. repeat same steps for `slf4j-simple` package and set scope to `implementation` instead of `testImplementation`
1. create a new package, e.g., `com.github.alainbouchard.kafka-demo.demo1`

#### Create a simple producer

The following are needed when creating a producer:

- kafka `producer` properties (or `Properties` object)
- kafka `producter` creation
- Send data (verification)
- Close the `producer`

Reference for [Kafka Producer Properties](https://kafka.apache.org/documentation/#producerconfigs).

```java
public class KafkaProducerDemo {

    public static void main(String[] args) {
        Properties properties = new Properties();

        // Set Producer Properties
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:29092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // Create Kafka Producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        ProducerRecord<String, String> record = new ProducerRecord<>("first_topic", "hello world from Java!");

        // Send data
        producer.send(record); // asynchronous, need to flush data
        producer.flush();

        // Tear down Producer
        producer.close();
    }
}
```

#### Create a producer with a callback

```java
public class KafkaProducerDemoWithCallback {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(KafkaProducerDemoWithCallback.class);

        Properties properties = new Properties();

        // Set Producer Properties
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:29092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // Create Kafka Producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        ProducerRecord<String, String> record = new ProducerRecord<>("first_topic", "hello world from Java!");

        // Send data
        producer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                // execute everytime a message is sent OR an exception is thrown.
                if (exception == null) {
                    // successfully sent message
                    logger.info("Received metadata - Topic: "
                            + metadata.topic() + " Partition: "
                            + metadata.partition() + " Offset: "
                            + metadata.offset() + " Timestamp: "
                            + metadata.timestamp());
                } else {
                    logger.error("Error while producer sent a message", exception);
                }
            }
        }); // asynchronous, need to flush data
        producer.flush();

        // Tear down Producer
        producer.close();
    }
}
```

#### Create a simple consumer

The following are needed when creating a `consumer`:

- kafka `consumer` properties (or `Properties` object)
- kafka `consumer` creation
- poll data in a loop

Reference for [Kafka Consumer Properties](https://kafka.apache.org/documentation/#consumerconfigs).

```java
public class KafkaConsumerDemo {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(KafkaConsumerDemo.class.getName());

        // Configure the consumer
        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:29092");
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "my_group");
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // Possible values : none, earliest, latest

        // Create consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        // Subscribe the consumer to the Topic or Topics
        consumer.subscribe(Arrays.asList("first_topic"));

        // Poll for new data
        while(true) { // Avoid in production - demo purpose only.
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

            records.forEach(r -> logger.info("Key: " + r.key()
                        + " Value: " + r.value()
                        + " Partition: " + r.partition()
                        + " Offset: " + r.offset()
                        + " Timestamp: " + r.timestamp()));
        }
    }
}
```

#### Create a consumer in a thread

Note: It is only working if the application does stop gracefully.  A Break/Kill signal won't trigger the shutdown properly.  IntelliJ don't have the `exit` button on Windows.

```java

public class KafkaConsumerWithThreadsDemo {
    Logger logger = LoggerFactory.getLogger(KafkaConsumerWithThreadsDemo.class.getName());

    public KafkaConsumerWithThreadsDemo() { }

    public void run() {
        String bootstrapServer = "localhost:29092";
        String groupId = "my_group";
        String topic = "first_topic";

        // Latch for dealing with multiple threads;
        CountDownLatch latch = new CountDownLatch(1);

        // Create Consumer Runnable;
        Runnable consumerRunnable = new ConsumerRunnable(bootstrapServer, groupId, topic, latch);

        // Starting Consumer Runnable Thread;
        Thread thread = new Thread(consumerRunnable);
        thread.start();

        // Add a shutdown hook;
        Runtime.getRuntime().addShutdownHook(new Thread( () -> {
            logger.info("Received a shutdown hook...");
            ((ConsumerRunnable) consumerRunnable).shutdown();

            try {
                latch.await();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

            logger.info("Consumer application has exited...");
        }));

        try {
            latch.await();
        } catch (InterruptedException e) {
            logger.error("Consumer application got interrupted", e);
        } finally {
            logger.info("Consumer application is closing...");
        }

    }

    public class ConsumerRunnable implements Runnable {
        private CountDownLatch latch;
        KafkaConsumer<String, String> consumer;

        public ConsumerRunnable(String bootstrapServer, String groupId, String topic, CountDownLatch latch) {
            this.latch = latch;

            // Configure the consumer
            Properties properties = new Properties();
            properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
            properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
            properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
            properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId);
            properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // Possible values : none, earliest, latest

            // Create consumer
            consumer = new KafkaConsumer<String, String>(properties);

            // Subscribe the consumer to the Topic or Topics
            consumer.subscribe(Arrays.asList(topic));
        }

        @Override
        public void run() {
            // Poll for new data
            try {
                while (true) { // Avoid in production - demo purpose only.
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

                    records.forEach(r -> logger.info("Key: " + r.key()
                            + " Value: " + r.value()
                            + " Partition: " + r.partition()
                            + " Offset: " + r.offset()
                            + " Timestamp: " + r.timestamp()));
                }
            } catch ( WakeupException exception) {
                logger.info("Received shutdown signal...");
            } finally {
                consumer.close();
                latch.countDown(); // telling caller code that this thread is done.
            }
        }

        public void shutdown() {
            // to interrupt the consumer.poll() method

            // and will make consumer.poll() to throw an exception WakeupException
            consumer.wakeup();
        }
    }

    public static void main(String[] args) {
        new KafkaConsumerWithThreadsDemo().run();
    }
}
```

#### Assign and seek consumer

The Assign and Seek is mostly used to replay data or fetch a specific message.

```java
public class KafkaConsumerWithAssignAndSeekDemo {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(KafkaConsumerWithAssignAndSeekDemo.class.getName());

        // Configure the consumer
        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:29092");
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "my_group"); No needs for group with assign and seek...
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // Possible values : none, earliest, latest

        // Create consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        // The Assign and Seek is mostly used to replay data or fetch a specific message.

        // Assign
        TopicPartition partition = new TopicPartition("first_topic", 0);
        consumer.assign(Arrays.asList(partition));

        // Seek
        consumer.seek(partition, 1L);

        // Poll for new data
        while(true) { // Avoid in production - demo purpose only.
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

            records.forEach(r -> logger.info("Key: " + r.key()
                        + " Value: " + r.value()
                        + " Partition: " + r.partition()
                        + " Offset: " + r.offset()
                        + " Timestamp: " + r.timestamp()));
        }
    }
}
```

#### Kafka Bidirectional Compatibility

As Kafka 0.10.2 (July 2017), the client and brokers hava a vapability called bi-directional compatibility (because API calls are nov versioned).

It means that __the latest client library version should always be used__ as documented in the confluent documentation [Upgrading Apache Kafka Clients Just Got Easier](https://www.confluent.io/blog/upgrading-apache-kafka-clients-just-got-easier/)

#### Creating a Producer for Twitter messages

Needed packages:

```groovy
dependencies {
    implementation group: 'com.twitter', name: 'twitter-api-java-sdk', version: '1.2.4'
}
```

Creating the topic:

```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic twitter_tweets --partitions 6 --replication-factor 1
```

Adding a `TwitterKafkaProducerInterface` Interface:

```java
package com.github.alainbouchard.kafka_demo.demo2;

public interface TwitterKafkaProducerInterface {
    void send(String topic, String message);
}
```

Implementing the interface with `TwitterKafkaProducer` Class:

```java
public class TwitterKafkaProducer implements TwitterKafkaProducerInterface {
    private final Logger logger = LoggerFactory.getLogger(TwitterKafkaProducer.class.getName());
    private KafkaProducer<String, String> producer;

    public TwitterKafkaProducer() {
        Properties properties = new Properties();

        // Set Producer Properties
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:29092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // Create Kafka Producer
        producer = new KafkaProducer<String, String>(properties);

        // Create topic:
        // kafka-topics --bootstrap-server localhost:9092 --create --topic twitter_tweets --partitions 6 --replication-factor 1
    }

    public Logger getLogger() {
        return logger;
    }

    @Override
    public void send(String topic, String message) {
        ProducerRecord producerRecord = new ProducerRecord<>(topic, null, message);
        producer.send(producerRecord, new Callback() {
            @Override
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                if (exception != null) {
                    getLogger().error("Could not send the message to the producer.", exception);
                }
            }
        });
    }
}
```

Adding the `TwitterListerner` Class:

Some TwitterApi v2 SDK methods have issues at the moment that this document is written.

```Java
public class TwitterListener {
    /***
     * Ref:
     *    https://developer.axonivy.com/api-browser?url=/market-cache/twitter/twitter-connector-product/9.3.0/openapi.json#/Tweets/addOrDeleteRules
     *    https://github.com/twitterdev/twitter-api-java-sdk/tree/d0d6a8ce8db16faf4e3e1841c3a43bd5a56aa069
     *    https://developer.twitter.com/en/docs/twitter-api
     */

    private Logger logger = LoggerFactory.getLogger(TwitterListener.class.getName());
    // API V2 uses BEARER token.
    // TODO: Use environment variable for BEARER_TOKEN.
    private final String BEARER_TOKEN = "";
    protected TwitterApi twitterApi;

    private TwitterKafkaProducerInterface twitterKafkaProducer;

    Function<List<Rule>, List<String>> GetIdsFromRules = r -> r.stream().map(Rule::getId).collect(Collectors.toList());

    public TwitterListener() {
        twitterApi = new TwitterApi();
        TwitterCredentialsBearer credentials = new TwitterCredentialsBearer(BEARER_TOKEN);

        twitterApi.setTwitterCredentials(credentials);
    }

    public Logger getLogger() {
        return logger;
    }

    public TwitterApi getTwitterApi() {
        return twitterApi;
    }

    public void setTwitterKafkaProducer(TwitterKafkaProducerInterface twitterKafkaProducer) {
        this.twitterKafkaProducer = twitterKafkaProducer;
    };

    private void logApiExceptionToString(String description, ApiException e) {
        String text = description
                + " Status code: " + e.getCode()
                + " Reason: " + e.getResponseBody()
                + " Response headers: " + e.getResponseHeaders();
        getLogger().error(text, e);
    }

    private List<Rule> addRule(String value, String tag, boolean dryRun) {
        // Create rule
        RuleNoId ruleNoId = new RuleNoId();
        ruleNoId.setValue(value);
        ruleNoId.setTag(tag);

        // Add rule to list of rules
        List<RuleNoId> ruleNoIds = new ArrayList<>();
        ruleNoIds.add(ruleNoId);

        // Add the list of rules to the request
        AddRulesRequest addRulesRequest = new AddRulesRequest();
        addRulesRequest.add(ruleNoIds);

        AddOrDeleteRulesRequest addOrDeleteRulesRequest = new AddOrDeleteRulesRequest();
        addOrDeleteRulesRequest.setActualInstance(addRulesRequest);

        List<Rule> rules = null;

        try {
            AddOrDeleteRulesResponse result = getTwitterApi().tweets().addOrDeleteRules(addOrDeleteRulesRequest, dryRun);
            getLogger().info(result.toString());

            rules = result.getData();
        } catch (ApiException e) {
            logApiExceptionToString("Exception when calling TweetsApi#addOrDeleteRules", e);
        }

        return rules;
    }

    private AddOrDeleteRulesResponse deleteRules(List<Rule> rules, boolean dryRun) {
        DeleteRulesRequestDelete deleteRulesRequestDelete = new DeleteRulesRequestDelete();
        deleteRulesRequestDelete.ids(GetIdsFromRules.apply(rules));

        DeleteRulesRequest deleteRulesRequest = new DeleteRulesRequest();
        deleteRulesRequest.setDelete(deleteRulesRequestDelete);

        AddOrDeleteRulesRequest addOrDeleteRulesRequestForDelete = new AddOrDeleteRulesRequest();
        addOrDeleteRulesRequestForDelete.setActualInstance(deleteRulesRequest);

        AddOrDeleteRulesResponse result = null;

        try {
            result = this.getTwitterApi().tweets().addOrDeleteRules(addOrDeleteRulesRequestForDelete, dryRun);
            getLogger().info(result.toString());
        } catch (ApiException e) {
            logApiExceptionToString("Exception when calling TweetsApi#addOrDeleteRules", e);
        }

        return result;
    }

    private GetRulesResponse getRules(String paginationToken, List<Rule> rules, Integer maxResults) {
        List<String> ruleIds = rules != null? GetIdsFromRules.apply(rules) : null;
        GetRulesResponse result = null;

        try {
            result = getTwitterApi().tweets().getRules(ruleIds, maxResults, paginationToken);
            getLogger().info(result.toString());
        } catch (ApiException e) {
            logApiExceptionToString("Exception when calling TweetsApi#getRules", e);
        }

        return result;
    }

    private GetRulesResponse getRules(String paginationToken, List<Rule> rules) {
        return this.getRules(paginationToken, rules, 1000);
    }

    private GetRulesResponse getRules(String paginationToken) {
        return this.getRules(paginationToken, null);
    }

    private void searchStream() {
        Set<String> expansions = new HashSet<>(Arrays.asList());
        Set<String> tweetFields = new HashSet<>();
        tweetFields.add("id");
        tweetFields.add("author_id");
        tweetFields.add("created_at");
        tweetFields.add("text");
        Set<String> userFields = new HashSet<>(Arrays.asList());
        Set<String> mediaFields = new HashSet<>(Arrays.asList());
        Set<String> placeFields = new HashSet<>(Arrays.asList());
        Set<String> pollFields = new HashSet<>(Arrays.asList());
        Integer backfillMinutes = null;  // There is a bug in the Twitter API v2 where any specified value will cause an error.

        InputStream result = null;

        try {
            result = getTwitterApi().tweets().searchStream(expansions, tweetFields, userFields, mediaFields, placeFields, pollFields, backfillMinutes);

            try {
                JSON json = new JSON();
                Type localVarReturnType = new TypeToken<FilteredStreamingTweet>(){}.getType();
                BufferedReader reader = new BufferedReader(new InputStreamReader(result));
                String line = reader.readLine();

                while (line != null) {
                    if (line.isEmpty()) {
                        getLogger().info("==> Empty line");
                        line = reader.readLine();
                        continue;
                    }
                    Object jsonObject = json.getGson().fromJson(line, localVarReturnType);
                    String message = jsonObject != null ? jsonObject.toString() : "Null object";
                    getLogger().info(message);
                    twitterKafkaProducer.send("twitter_tweets", message);

                    line = reader.readLine();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

        } catch (ApiException e) {
            logApiExceptionToString("Exception when calling TweetsApi#searchStream", e);
        }
    }

    public static void main(String[] args) {
        TwitterListener twitterListener = new TwitterListener();
        twitterListener.setTwitterKafkaProducer(new TwitterKafkaProducer());

        boolean dryRun = false;

        // Delete all existing Rules;
        try {
            GetRulesResponse rulesResponse = twitterListener.getRules(null);
            AddOrDeleteRulesResponse result = twitterListener.deleteRules(rulesResponse.getData(), dryRun);
            twitterListener.getLogger().info("Deleted rules: " + twitterListener.GetIdsFromRules.apply(result.getData()));
        } catch (Exception e) {
            twitterListener.getLogger().error("oops!");  // bug in the TwitterApi SDK.
            e.printStackTrace();
        }

        // Adding Rules;
        twitterListener.addRule( "potus -is:retweet", "Non-retweeted potus tweets", dryRun);
        twitterListener.addRule( "hockey -is:retweet", "Non-retweeted hockey tweets", dryRun);
        twitterListener.addRule( "baseball -is:retweet", "Non-retweeted baseball tweets", dryRun);
        twitterListener.addRule( "bitcoin -is:retweet", "Non-retweeted bitcoin tweets", dryRun);

        // Filter twitter stream;
        twitterListener.searchStream();
    }
}
```

#### Fine-tuning the Kafka producer

##### Producers Acks

###### acks=0

- no response is requested
- may loose data if broker goes offline
- it is okay when lost of data is acceptable:
  - metrics collection
  - log collection

```text
Producer             Broker 101 partition 0 (leader)
--------             -------------------------------
    |                               |
    +-----[send data to leader]---->+
    |                               |
```

###### acks=1 (default as Kafka v2.0)

- leader response is requestion (no guarantee of replication)
- the producer may retry if the tack isn't received
- data loss is possible if `leader broker` goes offline and `replcas` haven't replicated the data yet

```text
Producer             Broker 101 partition 0 (leader)
--------             -------------------------------
    |                               |
    +----[send data to leader]----->+
    |                               |
    +<---[respond write reqs]-------+
    |                               |
```

###### acks=all (replicas acks)

- leader and replicas acks are requested
- adding latency and safety
- no data loss if enough replicas
- __needed setting to avoid losing data__

```text
Producer             Broker 101 part0 (leader)    Broker 102 part0 (replica)    Broker 103 part0 (replica)
--------             -------------------------    --------------------------    --------------------------
    |                            |                            |                            |
    +---[send data to leader]--->+                            |                            |
    |                            |                            |                            |
    |                            +-----[send to replica]----->+                            |
    |                            |                            |                            |
    |                            +-----[send to replicas]--------------------------------->+
    |                            |                            |                            |
    |                            +<--------[ack write]--------+                            |
    |                            |                            |                            |
    |                            +<--------[ack write]-------------------------------------+
    |                            |                            |                            |
    +<---[respond write reqs]----+                            |                            |
    |                            |
```

##### min.insync.replicas

- the `acks=all` must be used along with `min.insync.replicas`
- it can be set at the proker or topic level (override)
- `min.insync.replicas=2` means that at least 2 brokers that are ISR (incuding leader) must do the write aknowledgement
  - e.g., with `replication.factor=3`, the `min.insync.replicas=2` and `acks=all` then only 1 broker can go down or the producer will receive an exception on the `send` operation

##### Producer retries

- developers are expected to handle the exceptions or the data may be lost:
  - e.g., transcient failure: `NotEnoughReplicasException`
- a `retries` setting is available:
  - default is `0`
  - no limits to the retries, i.e., `Interger.MAX_VALUE`
  - in case of retries, there is a change that the messages will be sent in wrong order
  - relying on key-based ordering may be an issue
  - `max.in.flight.requests.per.connection` (default=5) can be used to help fixing message ordering issue, where `max.in.flight.requests.per.connection=1` will ensure ordering, and slow down the throughput.
  - Idempotent producer can be used to help with ordering issues if Kafka version is >= 1.0.0

##### Idempotent producer

- using `kafka >= 0.11` - an idempotent producer can be defined, which will solve the duplicates due to network issues (i.e., if ack is lost on the network)
- idempotent producers are great to guarantee a stable and safe pipeline
- it comes with when using `producerProps.put("enable.idempotence", true)`:
  - `retries=Integer.MAX_VALUE`
  - `max.in.flight.requests=1` with Kafka >= 0.11 and < 1.1
  - `max.in.flight.requests=5` with Kafka >= 1.1 for higher performance
  - `acks=all`
- The `min.insync.replicas=2` must also be specified since `enable.idempotence=true` property doesn't imply this configuration
- Note: running _safe producer_ might impact the throughput or lantency, and therefore the `use case` must be verified to make sure the producer is within the NFR expectations

An example of Producer settings to improve safeness:

```java
        // Make a safer producer
        properties.setProperty(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
        properties.setProperty(ProducerConfig.ACKS_CONFIG, "all");  // default value with Idempotence=true
        properties.setProperty(ProducerConfig.RETRIES_CONFIG, Integer.toString(Integer.MAX_VALUE)); // default value with Idempotence=true
        properties.setProperty(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "5"); // with kafka >= 2.0, otherwise use 1.
```

##### Message compression

Some compression benchmarks can be found the this blog: [Squeezing the firehose: getting the most from Kafka compression](https://blog.cloudflare.com/squeezing-the-firehose/)

- producer usually send data text-based (json) messages
- compression is set at the producer configuration only no needs for consumer or broker configuration
- the `compression.type` can be `none` (default), `gzip`, `lz4` and `snappy`
- compression will improve the throughput and performance

```text
+----------------+       +--------------------+      +---------------+
| Producer Batch |------>| Compression of the |----->| Kafka Cluster |
+----------------+       | batch of messages  |      +---------------+
                         +--------------------+
```

Compression advantages:

- smaller producer request size (compression up to 4x)
- faster to tranfer data over the network
- better throughput
- better disk utilisation in Kafka cluster

Compression disavantages:

- producers and consumers must commit CPU time for compression/decompression

Overall:

- snappy and lz4 are optimal for speed/compression ratio
- recommendations are to try the algorithms in a given use case and environment
- always use compression in prod
- should use along with `linger.ms` and `batch.size` configuration settings

##### Configuration: linger.ms and batch.size

- Kafka producer default behavior is to send records as soon as possible
  - it will have up to 5 requests in flight (batch)
  - batching messages are done simultaneously with messages are in-flight (no time wasted)
- smart batching allows kafka to increase throughput while maintaining very low latency
- `linger.ms` is the number of ms (default=0) that the producer is willing to wait to fully get a batch of messages
  - by introducing some lag (e.g., linger.ms=5) then we increase the cahnges of messages being sent together in a batch
  - at the cost of introducing a small delay (e.g., 5 ms) then the throughput can be increased, the compression is more efficient and the producer efficiency is better
- `batch.size` is the maximum mumber of bytes (default=16KB) that will be included in a batch
- increasing a batch size to higher number (32KB or 64KB) can help increasing the ompression, throughput, and efficiency of requests
- any message tha is bigger than a batch size will not be batched
- the producer will make or allocate a batch per partition
- the average batch size metric can be monitored by using the Kafka Producer Metrics

An example of Producer settings to improve efficiency:

```java
        // Improve throughput efficiency of the producer
        properties.setProperty(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
        properties.setProperty(ProducerConfig.LINGER_MS_CONFIG, "20");
        properties.setProperty(ProducerConfig.BATCH_SIZE_CONFIG, "32768");  // 32 KB
```

##### Producer default partition and key hashing

- The default key are hashed using `murmur2` algorithm
- it is possible - but maybe not suggested - to override the partitioner behavior using `partitioner.class`
- the formula from Kafka code: `targetPartition = Utils.abs(Utils.murmur2(record.key())) % numPartitions;`
- therfore the same key will _always_ go to the same partition
- changing the number of partition will cause key vs partition issues and should be avoided

##### Max.blocks.ms and buffer.memory

These are advanced settings and it is probably better to avoid tweaking them unless necessary.

- when the producer prodeuces faster tahn the broker can handle then the records will be memory buffered
- the `buffer.memory` is the size of the buffer (default is 32MB)
- a full buffer (e.g., full 32MB) will cause the `producer.send()` method to block (or wait)
- the `max.block.ms` (default=60000ms) is the waiting time befre the producer.send() method throw an exception, and causes are:
  - the producer has filled up the buffer
  - the broker is not accepting any new data
  - 60s has elapsed
- An exception may mean that the broker is down or overloaded as it can't handle the requests

#### Elastic Search

##### Adding elasticsearch docker container

The following must be added in order to add an `elastic search` container or server to the kafka-cluster:

docker-compose.yaml:

```yaml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.2
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      discovery.type: single-node
      node.name: es01
      cluster.name: kafka-cluster
```

To start the service from docker-compose:

```bash
> docker-compose up elasticsearch -d
> docker-compose ps

NAME                            COMMAND                  SERVICE             STATUS              PORTS
kafka-cluster-elasticsearch-1   "/usr/local/bin/dock…"   elasticsearch       running             0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp
kafka-cluster-kafka-1           "/etc/confluent/dock…"   kafka               running             0.0.0.0:29092->29092/tcp
kafka-cluster-zookeeper-1       "/etc/confluent/dock…"   zookeeper           running             0.0.0.0:22181->2181/tcp

> curl localhost:9200/

{
  "name" : "es01",
  "cluster_name" : "kafka-cluster",
  "cluster_uuid" : "-w05UbWdSeOhc4nRGcy8yA",
  "version" : {
    "number" : "7.5.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8bec50e1e0ad29dad5653712cf3bb580cd1afcdf",
    "build_date" : "2020-01-15T12:11:52.313576Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

##### Java elasticsearch client

The following packages are required:

```groovy
    implementation group: 'org.elasticsearch.client', name: 'elasticsearch-rest-high-level-client', version: '7.14.0'
    implementation group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.17.2' // Elastic search dependency
```

```java
public class ElasticsearchClient {
    Logger logger = LoggerFactory.getLogger(ElasticsearchClient.class.getName());

    private RestHighLevelClient restHighLevelClient;

    public ElasticsearchClient() {
        restHighLevelClient = new RestHighLevelClient(
                // TODO: use configuration file or environment variables to set ip and ports, now configured for local docker containers.
                RestClient.builder(new HttpHost("localhost", 9200, "http"),
                        new HttpHost("localhost", 9300, "http")));
    }

    public RestHighLevelClient getRestHighLevelClient() {
        return restHighLevelClient;
    }

    public boolean ping() {
        boolean result = false;
        try {
            result = getRestHighLevelClient().ping(RequestOptions.DEFAULT);
        } catch (IOException e) {
            logger.error("Elasticsearch client received an exception.", e);
        } finally {
            logger.info("Elasticsearch aliveness: " + result);
        }

        return result;
    }

    public IndexResponse toIndex(String index, String jsonSource, String id) {
        IndexRequest indexRequest = new IndexRequest(index)
                .id(id) // Make the entry idempotent.
                .source(jsonSource, XContentType.JSON);

        IndexResponse indexResponse = null;
        try {
            indexResponse = getRestHighLevelClient().index(indexRequest, RequestOptions.DEFAULT);
            logger.info(indexResponse.getId());
        } catch (IOException e) {
            logger.error("Caught and exception.", e);
        }

        return indexResponse;
    }

    public void close() {
        try {
            getRestHighLevelClient().close();
        } catch (IOException e) {
            logger.error("Caught and exception.", e);
        }
    }

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(ElasticsearchClient.class.getName());

        // Setup Elasticsearch
        ElasticsearchClient elasticsearchClient = new ElasticsearchClient();

        elasticsearchClient.ping();

        // Setup Kafka Consumer
        TwitterKafkaConsumer twitterKafkaConsumer = new TwitterKafkaConsumer();
        twitterKafkaConsumer.subscribe("twitter_tweets");

        // Get data using Kafka Consumer and insert data to the elasticsearch
        while(true) { // TODO: replace with better logic.
            ConsumerRecords<String, String> records = twitterKafkaConsumer.poll(Duration.ofMillis(100));

            records.forEach(record -> {
                FilteredStreamingTweetResponse tweet = twitterKafkaConsumer.mapTweetToObject(record.value());
                elasticsearchClient.toIndex("twitter", record.value(), tweet.getData().getId());

                logger.debug("Key: " + record.key()
                        + " Value: " + record.value()
                        + " Partition: " + record.partition()
                        + " Offset: " + record.offset()
                        + " Timestamp: " + record.timestamp());

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    logger.error("Error while waiting for sleep", e);
                }
            });
        }

        // Tear-down the Elasticsearch
//        elasticsearchClient.close();

        // Tear-down the Kafka Consumer
    }
}
```

#### Kafka delivery semantics

The `at least once` processing should used for most applications along with `idempotent` strategy.

##### At most once

Offsets are committed as soon as the message batch is received. If the processing oes wrong, the message will be lost (it won't be read again)

##### At least once

Offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can resilt in duplicate processing of messages. Idempotence of the messages is needed to avoid duplicates.

##### Exactly once

Can be achieved for kafka-to-kafka workflows using kafka streams APIs. For the Kafka-to-Sink workflows, idempotent consumer is needed.

#### Idempotentce and offset auto-commit

The default configuration (`ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG="enable.auto.commit"`) setting is `at-least-once` if not specified otherwise.

To avoid duplicate entries, the usage of a unique key will be required. An example would be the Twitter ID.

#### Consumer Poll Behaviours

Default values should be correct in most cases but the following can be adjuested.

- `fetch.min.bytes` (default=1):
  - controls the minimum data to pull on each request
  - help improving throughput and decreasing request numbr
  - cost is latency

- `max.poll.records` (default=500):
  - controls the number of records to receive per poll request
  - increase if the messages are very small and if RAM is available
  - best practices tell to monitor the number of records per poll request and to adjust to increase the value if default value is often reached

- `max-partitions.fetch.bytes` (default=1MB):
  - maximum data returned by the broker per partition
  - reading from many partions will require a lot of memory

- `fetch.max.bytes` (default 50MB):
  - max data returned for each fetch request (covers multiple partitions)
  - the consumer performs multiple fetches in parallel

#### Consumer Offset Commits Strategies

The two strategies:

- `enable.auto.commit=true` and syncrhonous processing of record batches (default)
  - pseudocode:

    ```java
    while(true) {
        List<Records> records = consumer.poll(Duration.ofMillis(100));
        doSomethingSunchronous(records);
    }

    ```

    - offsets get automatically commited at regular interval `auto.commit.interval.ms=5000` (default)
    - Note: using asyncrhonous processing would make the delivery sementic to `at-most-once` since the offset will be committed before the data is processes

- `enable.auto.commit=false` and manual commit of offsets
  - pseudocode:

    ```java
    while(true) {
        records += consumer.poll(Duration.ofMillis(100));
        if isReady(records) {
            doSomethingSynchronous(records);
            consumer.commitSync();
        }
    }
    ```

  - offsets commit is controlled according to the expected conditions
  - E.g., accumulation recortds into a buffer and then flushing the buffer to a DB, then offsets are committed

##### Manual commits

Settings to be configured to avoid offsets auto commits:

```java
properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,"false"); // Will require manual offsets commits
properties.setProperty(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "10"); // Only retrieve 10 records at the time
```

The syncronous commit command for the consumer - pseudocode:

```java
while(true) {
    ConsumerRecords<String, String> records = twitterKafkaConsumer.poll(Duration.ofMillis(100));

    logger.info(String.format("Received %s records", records.count()));
    records.forEach(record -> {
        // do something synchronous here...
    });
    logger.info("Committing offsets");
    twitterKafkaConsumer.commitSync();
    logger.info("Offsets were committed");
}
```

Using `kafka-console-consumer` command:

- looking at the offsets for given group `kafka-java-demo-elasticsearch`:

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --group kafka-java-demo-elasticsearch --describe

Consumer group 'kafka-java-demo-elasticsearch' has no active members.

GROUP                         TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
kafka-java-demo-elasticsearch twitter_tweets  3          0               39              39              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  0          0               37              37              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  4          0               63              63              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  5          0               43              43              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  2          0               37              37              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  1          0               58              58              -
 -               -
sh-4.4$
```

- consuming records using `Java` consumer and verifying the offsets:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --group kafka-java-demo-elasticsearch --describe

Consumer group 'kafka-java-demo-elasticsearch' has no active members.

GROUP                         TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
kafka-java-demo-elasticsearch twitter_tweets  3          0               39              39              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  0          0               37              37              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  4          0               63              63              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  5          30              43              13              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  2          0               37              37              -
 -               -
kafka-java-demo-elasticsearch twitter_tweets  1          0               58              58              -
 -               -
sh-4.4$
```

- Note: the current offset for the partition 5 (above example) should be a multiple of `ConsumerConfig.MAX_POLL_RECORDS_CONFIG` property value, any record (modulo) will be consumed again since the offset wasn't committed

#### Consumer offset reset behaviour

The offset reset available behaviours are:

- `auto.offset.reset=latest` which will read from the end of the log
- `auto.offset.reset=earliest` which will read from start of the log
- `auto.offset.reset=none` which will throw exception if no offset is found

The consumer offsets can be lost:

- if a consumer hasn't read new data for 1 day (kafka < 2.0)
- if a consumner hasn't read new data for 7 days (kafka >= 2.0)

The retention delay can be set using the broker setting `offset.retention.minutes`. Proper data and offset retention period must be set to ensure no data loss if a server can go down for a while, e.g., 30 days.

#### Replaying the data for a consumner group

To reset kafka server offsets and replay messages from beginning:

```bash
sh-4.4$ kafka-consumer-groups --bootstrap-server localhost:9092 --topic twitter_tweets --group kafka-java-demo-elasticse
arch --reset-offsets --to-earliest --execute

GROUP                          TOPIC                          PARTITION  NEW-OFFSET
kafka-java-demo-elasticsearch  twitter_tweets                 3          0
kafka-java-demo-elasticsearch  twitter_tweets                 0          0
kafka-java-demo-elasticsearch  twitter_tweets                 4          0
kafka-java-demo-elasticsearch  twitter_tweets                 5          0
kafka-java-demo-elasticsearch  twitter_tweets                 2          0
kafka-java-demo-elasticsearch  twitter_tweets                 1          0

sh-4.4$
```

This allows to restart the consumer and replay the messages; having an idempotent server will make replays safe.

#### Controlling Consumer Liveliness

- consumers in a `group` talk to a `Consumer Group Coordinator`
- there is a `heartbeat` and a `pool` mechanism to detect if consumers are down
- best practices tell that a process should process data fast and poll often

Consumer Heartbeat Thread:

- `session.timeout.ms` (default=10s)
  - heardbeets are sent periodically to broker
  - if no heartbeat is sent during that period, the consumer is considered dead
  - set low value to faster consumer rebalancing

- `heartbeat.interval.ms` (default=3s)
  - wait period between 2 heartbeats
  - best pratices tell to set 1/3rd of the `session.timeout.ms` value

- both settings are set together to detect a dead consumer application (down)

Consumer Poll Thread:

- `max.poll.interval.ms` (default=5m)
  - maximum time between 2 poll() alls before declaring the consumer is dead
  - relevant for Big Data frameworks (e.g., Spark) in case processing takes time

- mecahnism to detect a data processing issue with the consumer

### Kafka Connect and Kafka Stream

- There are 4 Kafka use cases:
  - Source to Kafka => Producer API => Kafka Connect Source
  - Kafka to Kafka => Consumer API/Producer API => Kafka Streams
  - Kafka => Sink => Consumer API => Kafka Connect Sink
  - Kafka => Application => Consumer API =>
- Kafka connect and Kafka stream will simplify and improve the in/out of Kafka
- Kafka connect and Kafka stream will simplify transforming data within kafka withou relying on external libraries

The Kafka connectors can be found on the [Confluent Kafka Connectors](https://www.confluent.io/product/confluent-connectors) page.

#### Why Kafka Connect?

- developpers always want to import data from the same sources: DB, JDBC, Couchbase, Goldergate, SAP HANA, Blockchain, Cassandra, DynamoDB, FTP, IOT, MongoDB, MQTT, RethinkDB, Salesforce, Solr, SQS, Twitter, etc.
- developpers always want to store data in the same sinks: S3, ElasticSearch, HDFS, JDBC, SAP HANA, DocumentDB, Cassandra, DynamoDB, HBase, MongoDB, Redis, Solr, Splunk, Twitter, etc.
- an _unexperimented developper_ may struggle to achieve fault tolerance, idempotence, distribution, ordering, etc.
- an _experimented developper_  already did the work for others.

```text
Sources/Destinations     Connect Cluster             Kafka Cluster          Stream Apps
--------------------     ---------------             -------------          -----------
     [source1]------------->[worker]                   [broker]-------------->[app1]
                            [worker]------------------>[broker]<--------------
     [source2]              [worker]                   [broker]
                            [worker]<------------------[broker]-------------->[app2]
      [sinks]<--------------[worker]                           <--------------

```

##### Kafka Connect: High level

- source connectors to get data from Common Data Sources
- sink connectors to publish data in common data stores
- make it easy for non-experienced developpers to quickly get data in kafka
- part of the ETL pipeline (Extract, Transform and Load)
- scaling made easy from small pipelines to company-wide pipelines
- re-usable code

##### Adding a connector to a Java project

The connector [kafka-connect-twitter](https://www.confluent.io/hub/jcustenborder/kafka-connect-twitter) will be used as an example to show file structure:

```text
+ [my-java-project]
+-+ [kafka-connect] (to be created with sub folders)
| +-+ [connectors]
| | +-+ [kafka-connect-twitter] (downloaded connector project from github)
| | | +-+ *.jar
| | +-+ connector-standalone.properties
| | | + run.sh
| | | + twitter.properties

```

##### Connect standalone CLI command

```bash
sh-4.4$ connect-standalone
USAGE: /usr/bin/connect-standalone [-daemon] connect-standalone.properties

sh-4.4$
```

Creating a new topic for the kafka connector:

```bash
$ kafka-topics --bootstrap-server localhost:9092 --create --topic twitter_status_connect --parti
tions 3 --replication-factor 1

WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.

Created topic twitter_status_connect.

$ kafka-topics --bootstrap-server localhost:9092 --create --topic twitter_deletes_connect --part
itions 3 --replication-factor 1

WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.

Created topic twitter_deletes_connect.
$
```
