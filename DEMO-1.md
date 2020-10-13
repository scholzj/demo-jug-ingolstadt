# Demo: Running Kafka, Console consumer and producer, management tools

## Download

* Download Apache Kafka 2.6.0 from [http://kafka.apache.org/downloads](http://kafka.apache.org/downloads)
* Unpack it into `kafka-2.6.0` directory
```
mkdir kafka-2.6.0
tar --strip-components=1 -C kafka-2.6.0 -xvf kafka_2.12-2.6.0.tgz
```

## Start Zookeeper and Kafka

* In different terminals:
    * Start Zookeeper
        * `./kafka-2.6.0/bin/zookeeper-server-start.sh ./kafka-2.6.0/config/zookeeper.properties`
    * Start Kafka
        * `./kafka-2.6.0/bin/kafka-server-start.sh ./kafka-2.6.0/config/server.properties`

## Consuming and producing messages

* Start the console producer:

```
./kafka-2.6.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
```

* Wait until it is ready (it should show `>`).
* Once ready, send some message by typing the message payload and pressing enter to send.
* Next we can consume the messages

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic
```

* But this will by default show only new messages. You can add `--from-beginning` to see the messages sent in the past:

```
./kafka-2.6.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning
```

## Management tools

### Topics

* List the topics:
```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

* Create new topic:
```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-topic --partitions 3 --replication-factor 1
```

* Describe them to in more detail:
```
./kafka-2.6.0/bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
```

## Kafka Connect

* Edit the file `config/connect-file-sink.properties` and change the `topics` field to `my-topic` and add the two following fields:

```
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.storage.StringConverter
```

* Run Kafka Connect with the file sink connector:

```
./kafka-2.6.0/bin/connect-standalone.sh ./kafka-2.6.0/config/connect-standalone.properties ./kafka-2.6.0/config/connect-file-sink.properties
```

* Send some messages to topic `connect-test`:

```
./kafka-2.6.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic connect-test
```

* Check the output file `test.sink.txt` to verify it got the messages:

```
cat test.sink.txt
```