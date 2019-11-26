# Presto/Kafka/Avro with Confluent Schema Registry

Presto currently does not support Confluent Schema Registry to decode Avro messages from 
Kafka. This repo is a proof-of-concept to test https://github.com/prestosql/presto/pull/2106.

## Preparing data in Kafka cluster

Start Kafka cluster:

```bash
docker-compose up -d
```

Ingest sample data:

```bash
$CONFLUENT_HOME/bin/kafka-avro-console-producer --broker-list localhost:9092 --topic t1 --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
{"f1": "value1"}
{"f1": "value2"}
```

Validate data is readable:

```bash
$CONFLUENT_HOME/bin/kafka-avro-ole-consumer --topic t1 --bootstrap-server localhost:9092 --from-beginning
{"f1":"value1"}
{"f1":"value2"}
```

## Querying data in Presto

> Confluent Schema Registry support for Presto is in development. Get a patched version from here: <https://github.com/jeqo/presto/releases/tag/326-SNAPSHOT>

Start Presto with provisioned configurations:

```
cp -r presto/etc $PRESTO_HOME/.
cd $PRESTO_HOME
bin/launcher run 
```

On another terminal check your tables:

```
$PRESTO_HOME/bin/presto
presto> select * from kafka.default.t1;
 key  | field  | _partition_id | _partition_offset | _segment_start | _segment_end | _segment_count | _message_corrupt | _message | _message_lengt
------+--------+---------------+-------------------+----------------+--------------+----------------+------------------+----------+---------------
 NULL | value1 |             0 |                 0 |              0 |            2 |              1 | false            | ^@^@^@^@^A^Lvalue1   |   
 NULL | value2 |             0 |                 1 |              0 |            2 |              2 | false            | ^@^@^@^@^A^Lvalue2   |   
(2 rows)
```