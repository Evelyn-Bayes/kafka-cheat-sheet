# A Kafka Cheat Sheet
## Command Line Tools
*Describe all topics*
```
kafka-topics --bootstrap-server localhost:9092 --describe
```

*Run console producer*
```
kafka-console-producer --broker-list localhost:9092 --topic <TOPIC>
```

*Run console producer with keys & values*
```
kafka-console-producer --broker-list localhost:9092 --property "parse.key=true" --property "key.separator=:" --topic <TOPIC>

Example input key:value
```

*Run producer performance test tool*
```
kafka-producer-perf-test --producer-props bootstrap.servers=localhost:9092 --topic <TOPIC> --throughput <THROUGHPUT> --num-records <NUM_RECORD> --record-size <SIZE>
```

*Consume topic from beginning*
```
kafka-console-consumer --bootstrap-server localhost:9092 --from-beginning --property "print.key=true" --topic <TOPIC>
```

*Alter topic configuration*
```
kafka-configs --bootstrap-server localhost:9092 --alter  --topic <TOPIC> --add-config <CONFIG>=<VALUE>
```

*Purge a topic*
```
kafka-topics --bootstrap-server localhost:9092 --alter --topic <TOPIC> --config retention.ms=1000
kafka-topics --bootstrap-server localhost:9092 --alter --topic <TOPIC> --config segment.ms=15000
#Wait for 30 second ...
kafka-topics --bootstrap-server localhost:9092 --alter --topic <TOPIC> --delete-config retention.ms
kafka-topics --bootstrap-server localhost:9092 --alter --topic <TOPIC> --delete-config segment.ms
```

## Zookeeper
*Open Zookeeper shell*
```
zookeeper-shell localhost:2181
```

*Run Zookeeper shell command to get controller ID*
```
zookeeper-shell localhost:2181 get /controller
```

*Run Zookeeper 'four letter word'*
```
echo srvr | ncat localhost 2181
```

## Kafka Connect

*Install connector from Confluent Hub*
```
confluent-hub install <CONNECTOR>
```

*Create connector*
```
curl -vX PUT http://localhost:8083/connectors/<NAME>/config --header "Content-Type: application/json" --data '{"connector.class": "<CLASS>", "tasks.max": "1", <INSERT-CONFIG-HERE>}'
```

*Create connector from file*
```
curl -vX POST http://localhost:8083/connectors --header "Content-Type: application/json" --data @/tmp/<JSON-FILE>'
```

*Delete connector*
```
curl -vX DELETE http://localhost:8083/connectors/<NAME>
```

*List all connectors*
```
curl -vX GET http://localhost:8083/connectors/
```

*Get connector status*
```
curl -vX GET http://localhost:8083/connectors/<NAME>/status
```

*Create file source connector*
```
curl -vX PUT http://localhost:8083/connectors/<NAME>/config --header "Content-Type: application/json" --data '{"connector.class": "FileStreamSource", "tasks.max": "1", "file": "/tmp/<FILE>", "topic": "<TOPIC>"}'
```

*Create file sink connector*
```
curl -vX PUT http://localhost:8083/connectors/<NAME>/config --header "Content-Type: application/json" --data '{"connector.class": "FileStreamSink", "tasks.max": "1", "file": "/tmp/<FILE>", "topics": "<TOPIC>"}'
```

## Schema Registry

*Create schema from file*
```
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/<SUBJECT>/versions --data @/tmp/<FILE>
```

*Create schema*
```
curl -vX POST -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/<SUBJECT>/versions --data '{"schema": "{\"type\": \"record\", \"name\": \"schema_name\", \"fields\": [{\"type\": \"string\", \"name\": \"string_field\"}, {\"type\": \"int\", \"name\": \"int_field\"}]}"}'
```

*Retrieve schema using its subject and version*
```
curl -X GET -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/<SUBJECT>/versions/<ID>
```

*Retrieve a list of schema versions for subject*
```
curl -X GET -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects/<SUBJECT>/versions
```

*Retrieve a list of subjects*
```
curl -X GET -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/subjects
```

*Retrieve schema using its ID*
```
curl -X GET -H "Content-Type: application/vnd.schemaregistry.v1+json" http://localhost:8081/schemas/ids/<ID>
```

*Run Json console producer*
```
kafka-json-schema-console-producer --broker-list localhost:9092 --property schema.registry.url=http://localhost:8081 --topic <TOPIC> --property value.schema='{"type":"object","properties":{"first-name":{"type":"string"}}}' --property key.schema='{"type":"object","properties":{"last-name":{"type":"string"}}}' --property parse.key=true --property key.separator=|

Example input {"last-name":"Bayes"}|{"first-name":"Evelyn"}
````

*Consume topic from beginning using Json schema*
```
kafka-json-schema-console-consumer --bootstrap-server localhost:9092 --property schema.registry.url=http://localhost:8081 --topic <TOPIC> --property value.schema='{"type":"object","properties":{"first-name":{"type":"string"}}}' --property key.schema='{"type":"object","properties":{"last-name":{"type":"string"}}}' --property print.key=true --from-beginning
```

*Dump schema topic file*
```
kafka-console-consumer --bootstrap-server kafka1:9092 --from-beginning --property print.key=true --timeout-ms 1000 --topic _schemas 1> schemas.log
```

*Restore schema topic from file*
```
kafka-console-producer --bootstrap-server kafka1:9092 --property parse.key=true --topic _schemas < schemas.log
```

## Rest Proxy

*List topics*
```
curl -X GET -H "Content-Type: application/vnd.kafka.v2+json" http://localhost:8082/topics
```

*Push records to Kafka using RestProxy*
```
curl -X POST -H "Content-Type: application/vnd.kafka.binary.v2+json" -H "Accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json" --data '{"records": [{"key": "<KEY>", "value": "<VALUE>", "partition": "<PARTITION>"}, {"value": "<VALUE2>"}]}' http://localhost:8082/topics/<TOPIC>
```

*Create RestProxy consumer group, subscribe to a topic and consume from it*
```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" http://localhost:8082/consumers/<CONSUMER_GROUP_ID>

curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics": ["topic1", "topic2"]}' http://localhost:8082/consumers/<CONSUMER_GROUP_ID>/instances/<INSTANCE>/subscription

curl -X GET -H "Content-Type: application/vnd.kafka.v2+json" http://localhost:8082/consumers/<CONSUMER_GROUP_ID>/instances/<INSTANCE>/records

```

## Confluent Server / Rest Proxy v3

*Print all clusters*
```
curl -X GET http://localhost:8082/v3/clusters/
```

*Print all ACLs*
```
curl -X GET http://localhost:8082/v3/clusters/<CLUSTER_ID>/acls
```

*Write ACL to cluster*
```
curl -X POST http://localhost:8082/v3/clusters/<CLUSTER_ID>/acls --header 'Content-Type: application/json' --data-raw '<DATA>'
```

## Usual Commands
*Consume offsets topic*
```
kafka-console-consumer --bootstrap-server localhost:9092 --from-beginning --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --topic __consumer_offsets
```

*List log levels for broker*
```
kafka-configs --bootstrap-server kafka1:9092 --describe --entity-type broker-loggers --entity-name <BROKER_ID>
```

*Set log level for class*
```
kafka-configs --bootstrap-server localhost:9092 --alter --add-config <LOGGER_CLASS>=<LEVEL> --entity-type broker-loggers --entity-name <BROKER_ID>
```

*Throttle incoming data*
```
kafka-configs --bootstrap-server localhost:9092 --entity-type brokers --entity-name <BROKER_ID> --add-config follower.replication.throttled.rate=<RATE> --alter

kafka-configs --bootstrap-server localhost:9092 --entity-type brokers --entity-name <BROKER_ID> --add-config follower.replication.throttled.replicas=* --alter

Note: This will only work if BROKER_ID is out of sync, otherwise it ignores the throttle
```

*Run Confluent Rebalancer*
```
confluent-rebalancer execute --bootstrap-server localhost:9092 --metrics-bootstrap-server localhost:9092 --throttle <RATE> --verbose
```

*Check status / finish Confluent Rebalancer*
```
confluent-rebalancer status --bootstrap-server localhost:9092
```

*Print JMX metrics to console*
```
kafka-run-class kafka.tools.JmxTool --object-name '<JMX_METRIC>' --jmx-url service:jmx:rmi:///jndi/rmi://<HOST>:<PORT>/jmxrmi --reporting-interval 1000

Example JMX_METRIC kafka.server:type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent
```

*Dump content of log, including batch sizes etc*
```
kafka-dump-log --deep-iteration --files /dir/<TOPIC>-<PARTITION>/<OFFSET_OF_LOG_FILE>.log
```
