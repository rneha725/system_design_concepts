## Overview
Kafka is [messaging system](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Messaging%20systems.md) based on pubsub, it can also be used a messaging queue, it is distributed, durable, fault-tolerant and highly scalable with zero downtime. Producers and consumers are fully decoupled, one does not need to wait for other.

## Applications

- It was first developed to process clickstream like data at linkedin.
- To store messages
- streaming real time data
- high throuput between entities by using consumer groups and let the multiple consumers consume from a partition.
- for metrics collection, log aggregation
- stream processing: Kafka is quite useful for use cases where the collected data undergoes processing at multiple stages. For example, the raw data consumed from a topic is transformed, enriched, or aggregated and pushed to a new topic for further consumption. This way of data processing is known as stream processing.
- used as commit log for distributed systems
- clickstream data

## Terminologies
- Brokers: System that stores the messages. 1:1 for broker and server. Cluster contains multiple servers.
- Record: kafka message unit, contains: key, value, timestamp(what is it?), and optional headers
- Topics: A broker can contain multiple topics and each topic can be subscribed by multiple consumers. One consumer can subscribe to multiple topics too. Also, the records are durable so they do not get deleted after consumption. They are stored till the specified amount of time or till the memory fills up.
- Zookeeper: it is read-optimized kv store, used to coordinate among brokers in a cluster. It stores the offset for consumer group's consumers. Stores the leader and followers for a record.ACLs.
- Parition: A topic is divided into partitions, for better performance and scale. A topic's partition can reside on multiple brokers. A topic's partitions can be read parallely. A partition maintains a offset for each message it receives. Paritions are also replicated and follow leader follower architecture.

    <img width="1327" alt="image" src="https://github.com/user-attachments/assets/e3a65b64-b911-47bc-a2af-3246a50607c5">
- Producer: producer of records
- Consumer[polls]: can subscribe to one or more topics. Consumers do not delete the messages, instead keep an offset with them. Any consumer can start polling from the previous offsets too. Consumer sends the ack to the broker and it saves the offset for the consumer in zookeeper.


## How it works?
- Commit log
- Leader-follower: For data management it uses leader follower mechnism, so each partition is also replicated to other broker. The info of leader-follower for a parition is stored in zookeeper, a producer/consumer directly connects to zookeeper for this info.
- In sync replicas and high water mark: An ISR(in-sync replica) always have the latest offset, a leader is always ISR, and the candidate when a leader fails is always one of the ISRs. High water mark is basically the highest minimum value present on all the replcas.
  <img width="797" alt="image" src="https://github.com/user-attachments/assets/cdb74f74-f38b-4c17-8503-e5a2b578b472">

  if in case a consumer, reads from the latest offset from the leader, on leader failure, the message won't be present and it will a non-repeatable read.
- Consumer groups: A consumer group is way to parallelize reading from a topic, using the paritions. There is a 1:n mapping between a conumer and partitions i.e. no two consumers in a consumer group can read from a single partition.
  <img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/50679d18-db14-4427-9fcf-f08e1669f341">
  number of consumers > number of paritions: some consumers will be idle and kafka might use them for failovers
  number of consumers < number of paritions: some consumers will be consuming from multiple partitions
  number of consumers = number of paritions: each consumer is consuming from unique partition
- Broker controller: There is one broker in the kafka cluster which performs the admin task of handling the introduction of a new broker, adding parititons, adding leaders, keep in checking the health of other brokers.
    - [Split brain](https://github.com/rneha725/system_design_concepts/blob/main/Systems/General%20issues%20in%20distributed%20systems.md#split-brain): if a controller dies, another is elected as the new one. But say the other one comes back as the new zombie controller. Then? this is a split brain issue where there are two dedicated service where only one central service is required.
    - Generational clock to solve this: each new broker is assigned a monotonically increasing value by using epoch time, other brokers only accept requests from the highest controller value. 

### Zookeeper
It is a distributed confuguration and synchronization service, it is read-optimized kv store, used to coordinate among brokers in a cluster. It stores the offset for consumer group's consumers. Stores the leader and followers for a record.ACLs. All the critical data is stored in the zookeeper cluster. Kafka producers, now, do not connect to zookeeper for the partition info for a key, but it happens through a call to any broker(not sure why?????)

![image](https://github.com/user-attachments/assets/fc4cb874-ee69-41c9-b49e-566088a0409b)





## Kafka Guarantees
- Records in a parition are always in sequence. Not across topics.
- Record related to one key always written to one partition.
## Pros and cons: tradeoffs

## Resources
