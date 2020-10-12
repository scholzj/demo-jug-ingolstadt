# Demo: Running Kafka, Console consumer and producer, management tools

## Download

* Download Apache Kafka from [http://kafka.apache.org/downloads](http://kafka.apache.org/downloads)
* Unzip it and enter the unzipped directory

## Start Zookeeper and Kafka

* In different terminals:
    * Start Zookeeper
        * `bin/zookeeper-server-start.sh config/zookeeper.properties`
    * Start Kafka
        * `bin/kafka-server-start.sh config/server.properties`

## Consuming and producing messages

* Start the console producer:

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
```

* Wait until it is ready (it should show `>`).
* Once ready, send some message by typing the message payload and pressing enter to send.
* Next we can consume the messages

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic
```

* But this will by default show only new messages. You can add `--from-beginning` to see the messages sent in the past:

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning
```

## Management tools

### Topics

* List the topics:
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

* Create new topic:
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-topic --partitions 3 --replication-factor 1
```

* Describe them to in more detail:
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
```

## Kafka Connect

* Edit the file `config/connect-file-sink.properties` and change the `topics` field to `my-topic` and add the two following fields:

```
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.storage.StringConverter
```

* Run Kafka Connect with the file sink connector:

```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-sink.properties
```

* Check the output file `test.sink.txt` to verify it got the messages:

```
cat test.sink.txt
```