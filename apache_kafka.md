# Apache Kafka

## Textbook Description
Apache Kafka is an open-source distributed event streaming platform used by
thousands of companies for high-performance data pipelines,
streaming analytics, data integration, and mission-critical applications.

## Downplayed Version

Smart storage service where events produced by one or multiple services are stored.
These events are then consumed by other services to perform some kind of processing or action.

```Note: An event is a piece of information in a string, JSON, or similar format.```

## Architecture
1. Kafka is a standalone/clustered application.
2. Kafka libraries communicate with this kafka servers using their own custom binary protocol over TCP network
3. Kafka Raft (KRaft) mode:
   - In communication between kafka brokers (nodes of the cluster), kafka used to zookeeper which essentially is a centralized service
   - this service chooses leader for the cluster, handles failuers, manager metadata, topics, partitions, handles node registation etc etc
   - Kafka RAFT (Kraft) is RAFT algorithm implemented for kafka.
   - since it's built in kafka itself and not a seperate service, we remove one layer from the arcitecture, which reduces latency and clusters also become more scalable
     
## Architecture Continued...
(insert diagram from https://www.projectpro.io/article/apache-kafka-architecture-/442)
1. Brokers: Kafka servers that store the data and serve clients.
2. Topics: Categories or feeds to which records/events are published. similar to a table in database.
3. Producers: Applications that publish (write) data to topics.
4. Consumers: Applications that subscribe to (read) data from topics.
5. Partitions: Sub-divisions of topics for parallelism and scalability.
6. Offsets: Unique IDs assigned to messages within a partition. incremental integer ID.

## Starting a kafka server (Kraft mode)

### config/kraft/controller.properties
```properties
node.id=1
process.roles=controller,broker
controller.quorum.voters=1@localhost:9093
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
log.dirs=/tmp/kraft-combined-logs
```

### starting broker with kraft
```sh
bin/kafka-server-start.sh config/controller.properties
```

### Creating a topic 
```sh
bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### Producing Messages
```sh
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092 <my message>
```

### Consuming Messages:
```sh
bin/kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --from-beginning
```

## Kafka with Java
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.7.0</version>
</dependency>
```

### Important APIs
1. Admin
2. Producer
3. Consumer

### Admin API
- You can manage Kafka topics programmatically using the AdminClient API.
```java
// Create Properties for admin client
Properties config = new Properties();
config.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

// create admin client with above properties
try (AdminClient admin = AdminClient.create(config)) {
  // create a new topic object
  NewTopic newTopic = new NewTopic("new-topic", 1, (short) 1);
  // pass list of topics to be created to admin client
  admin.createTopics(Collections.singleton(newTopic)).all().get();
  System.out.println("Topic created successfully");
} catch (Exception e) {
  e.printStackTrace();
}
```

### Producer API
```sh
// define producer properties including
// 1. server address
// 2. seriealizer classes for key and values
// etc
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

// create a producer
KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// create a ProducerRecord as the same type of producer (<String, String> here)
ProducerRecord<String, String> record = new ProducerRecord<>("test-topic", "key", "value");

try {
  // sending a producer record is async call which gives us a future object which we can evaluate to obtain
  // the metadata of a record like which partition is iot stored or what offset does the record have
  producer.send(record);
  System.out.println("Message sent successfully");
} finally {
  // close method is a blocking call that waits till all the records have been sent
  // then it calls flush internally and then closes the producer
  producer.close();
}
```
