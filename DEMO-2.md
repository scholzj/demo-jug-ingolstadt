# Demo: Topics, Partitions, Replication factor, ...

## Configs

### Kafka configuration

* Look at the Kafka configuration files in `./configs/`
* Notice:
    * `broker.id`
    * listeners, advertised listeners, protocols

## Start the Zookeeper and brokers

_(in different terminals)_

* Use the predefined files in `config` directory to start a multi-broker Kafka cluster
* Start Zookeeper
    * `./kafka-2.6.0/bin/zookeeper-server-start.sh config/zookeeper.properties`
* Start Kafka brokers
    * `./kafka-2.6.0/bin/kafka-server-start.sh config/server-0.properties`
    * `./kafka-2.6.0/bin/kafka-server-start.sh config/server-1.properties`
    * `./kafka-2.6.0/bin/kafka-server-start.sh config/server-2.properties`

## Basics

### Create topic

```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic demo --partitions 3 --replication-factor 3
```

### Check the created topic

```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list --topic demo
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic demo
```

Notice the distribution of leaders and the ISR replicas.

### Send some messages

* Send at least 10 messages (e.g. `Message 1`, `Message 2` etc. to be able to notice the ordering later)

```
./kafka-2.6.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic demo
```

### Consume messages

* Read from the whole topic

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning
```

* Notice how the messages are out of order. And check how nicely ordered they are in a single partition.

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --partition 0 --from-beginning
```

* Reading from a particular offset

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --partition 0 --offset 2
```

## Replication

### Broker crash

* Kill one of the brokers
* Look again at the topic description with the leaders which changed and new ISR

```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic demo
```

### Consume messages

* Try to consume the messages again to confirm that replication worked and that the messages are still in the topic!

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning
```

### Send some new messages

```
./kafka-2.6.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic demo
```

### Start the broker again

* Leadership didn't changed, but all replicas are again ISR

## Consumer Groups

### Create a new topic to get rid of the old messages

```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic demo
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic demo --partitions 3 --replication-factor 3
```

### Setup consumers

_(in different terminals)_

* Open 3 consumers using the same group `group-1`

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning --property print.key=true --property key.separator=":" --group group-1
```

* Open consumer using a different group `group-2`

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning  --property print.key=true --property key.separator=":" --group group-2
```

### Send messages

* Send some messages with keys (Send messages in the format `key:payload` - e.g. `my-key:my-value`)

```
./kafka-2.6.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic demo --property "parse.key=true" --property "key.separator=:"
```

### Rebalancing consumer group

* Kill one of the consumers started before
* Send some messages with the same key as was used before for this consumer
* Notice that one of the other consumers got the partition assigned and will receive it

### Message replay

* Consume the messages from Kafka with a new group:

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning  --property print.key=true --property key.separator=":" --group replay-group
```

* After it consumes all messages, try to restart it to make sure they were all committed - no messages should be received
* List all the groups:

```
./kafka-2.6.0/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --all-groups --list
```

* Or describe them:

```
./kafka-2.6.0/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --all-groups --describe
```

* Go and reset the offset to 0:

```
./kafka-2.6.0/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --reset-offsets --to-earliest --group replay-group --topic demo --execute
```

* Try to consume the messages again - you should receive them from the beginning of the topic:

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo --from-beginning  --property print.key=true --property key.separator=":" --group replay-group
```