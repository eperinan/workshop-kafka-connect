# workshop-kafka-connect

Enter in docker folder and run `docker-compose up` to run all the containers.

Check all worked fine `docker ps`

Enter in kafka connect and check the list of connectors `$ docker exec -i -t docker_kafka-connect_1  /bin/bash`

Load data `mysql -h 127.0.0.1 -u root -ppassword`

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

Enable jdbc connector and configure to connect to the previous database created.


```
curl -X POST   -H "Content-Type: application/json"   --data '{ "name": "quickstart-jdbc-source", "config": { "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", "tasks.max": 1, "connection.url": "jdbc:mysql://mysql:3306/connect_test?user=root&password=password", "mode": "incrementing", "incrementing.column.name": "id", "timestamp.column.name": "modified", "topic.prefix": "quickstart-jdbc-", "poll.interval.ms": 1000 } }'   http://localhost:8083/connectors

```

Verify the connector is installed properly

`curl -s localhost:8083/connector-plugins | jq .`


Describe topics in Kafka 

```
kafka-topics.sh  --zookeeper localhost:2181 --list
```
