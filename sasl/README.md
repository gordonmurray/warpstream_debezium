# CDC using Debezium and WarpStream with Simple Authentication and Security Layer (SASL)

Trying out WarpStream as a Kafka alternative with Debezium, using SASL.

Log in to the [WarpStream Console](https://console.warpstream.com/) and create a new Serverless cluster.

Once you create a Serverless cluster, you will be prompted to create access credentials. You'll get a username and password in the form of:

```
Username: ccun_000000000000000000000
Password: ccp_000000000000000000000
```

1. Update the `CONNECT_SASL_JAAS_CONFIG` environment variable with your username and password in the docker-compose.yml file attached.
2. Next, include the username/password in the connector at `files/connector.json`:

Start up the containers using:

```
docker-compose up -d
```

In the Warpstream console you'll see some initial topics created by Debezium.

To create a connector which will read the binlog of the mariadb container, use the following steps:

```
docker exec -it debezium /bin/bash
```

Create a Connector in Debezium:

```
curl -X PUT http://localhost:8083/connectors/my_database/config -H "Content-Type: application/json" -d @connector.json
```

You can confirm your connector was created by listing connectors:

```
curl localhost:8083/connectors
```

Check Connector status:

```
curl localhost:8083/connectors/my_database/status
```

Use the Kafka CLI to list topics in WarpStream:

First create a config called `config.properties` with the following content:

```
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="ccun_000000000000000000000" password="ccp_000000000000000000000";
```

Then you can list topics as follows:

```
./bin/kafka-topics.sh --bootstrap-server serverless.warpstream.com:9092 \
--command-config config.properties \
--list
```

List the messages in a topic

```
./bin/kafka-console-consumer.sh --bootstrap-server serverless.warpstream.com:9092 \
--consumer-property 'security.protocol=SASL_SSL' \
--consumer-property 'sasl.mechanism=PLAIN' \
--consumer-property 'sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username='"ccun_000000000000000000000"' password='"ccp_000000000000000000000"';' \
--topic testing.mydatabase.products
```