# workshop-kafka-connect

Enter in docker folder and run `docker-compose up` to run all the containers.

Check all containers are working fine: `docker ps`

Enter in the database and load the data: `mysql -h 127.0.0.1 -u root -ppassword`

```
CREATE DATABASE IF NOT EXISTS connect_test;
USE connect_test;

DROP TABLE IF EXISTS test;


CREATE TABLE IF NOT EXISTS test (
  id serial NOT NULL PRIMARY KEY,
  name varchar(100),
  email varchar(200),
  department varchar(200),
  modified timestamp default CURRENT_TIMESTAMP NOT NULL,
  INDEX `modified_index` (`modified`)
);

INSERT INTO test (name, email, department) VALUES ('alice', 'alice@abc.com', 'engineering');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
exit;
```

Enable jdbc as source connector and configure to connect to the previous database.

```
curl -X POST   -H "Content-Type: application/json" --data '{ 
    "name": "quickstart-jdbc-source4", 
    "config": { 
        "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", 
        "tasks.max": 1, 
        "connection.url": "jdbc:mysql://mysql:3306/connect_test?user=root&password=password", 
        "mode": "incrementing", 
        "incrementing.column.name": "id", 
        "timestamp.column.name": "modified", 
        "topic.prefix": "quickstart-jdbc-", 
        "poll.interval.ms": 1000 
    } 
}'   http://localhost:8083/connectors

```

Enable S3 as sink connector and configure to connect to publish each message to S3 bucket.

```
curl -X POST   -H "Content-Type: application/json"   --data '{ 
    "name": "quickstart-s3-sink", 
    "config": { 
        "connector.class": "io.confluent.connect.s3.S3SinkConnector", 
        "tasks.max": 1, 
        "topics" : "quickstart-jdbc-test",
        "s3.region": "eu-west-1",
        "s3.bucket.name" : "workshop-kafka-connect",
        "s3.part.size" : 5242880,
        "flush.size" : 3,
        "schema.compatibility":"NONE",
        "storage.class" : "io.confluent.connect.s3.storage.S3Storage",
        "format.class" : "io.confluent.connect.s3.format.avro.AvroFormat",
        "schema.generator.class" : "io.confluent.connect.storage.hive.schema.DefaultSchemaGenerator",
        "partitioner.class" : "io.confluent.connect.storage.partitioner.DefaultPartitioner"
                                                                 
    } 
}' http://localhost:8083/connectors

```

Verify the kafka connectors installed on the container.

`curl -s localhost:8083/connector-plugins | jq .`


Verify the connectors configured previously

`curl -s localhost:8083/connectors | jq .`

Describe topics available in Kafka.

```
kafka-topics.sh  --zookeeper localhost:2181 --list
```

Check messages in real time in the topic `quickstart-jdbc-test`:

`kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-jdbc-test --from-beginning`

Check files in S3 bucket

`aws s3 ls s3://workshop-kafka-connect/topics/quickstart-jdbc-test/partition=0/`