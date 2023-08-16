# WarpStream with Debezium
Trying out WarpStream as a Kafka alternative with Debezium. Read more about WarpStream from [Kafka is dead, long live Kafka](https://www.warpstream.com/blog/kafka-is-dead-long-live-kafka)

Install the Warpstream agent locally using

```
curl https://console.warpstream.com/install.sh | bash
```

Log in to the Warpstream console to get your pool name, Cluster ID and API key at [https://console.warpstream.com](https://console.warpstream.com)

Start the agent locally using the following command with an s3 bucket and WarpStream details added:

```
warpstream agent -agentPoolName apn_default -bucketURL mem://my-test-bucket -apiKey aks_xxxxxxxxxx -defaultVirtualClusterID vci_xxxxxxxxxx
```

Use Docker Compose to start Mariadb and Debezium containers to try to out CDC:

```docker-compose.yml
version: '3.1'
services:
  mariadb:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: test_db
    ports:
      - "3306:3306"

  debezium:
    image: debezium/connect:latest
    depends_on:
      - mariadb
    environment:
          BOOTSTRAP_SERVERS: api-xxxxxxxxxx.discovery.prod-z.us-east-1.warpstream.com:9092
          CONFIG_STORAGE_TOPIC: debezium_connect_config
          OFFSET_STORAGE_TOPIC: debezium_connect_offsets
    volumes:
      - ./debezium/connector.json/:/kafka/connector.json
```

Create an initial Connector JSON file for Debezium using the following JSON:

```
{
    "name": "connector",
    "config": {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "tasks.max": "1",
        "database.hostname": "mariadb",
        "database.port": "3306",
        "database.user": "root",
        "database.password": "root_password",
        "database.server.id": "12345",
        "database.server.name": "test_db",
        "database.whitelist": "test_db",
        "table.whitelist": "*",
        "database.history.kafka.bootstrap.servers": "localhost:9092",
        "database.history.kafka.topic": "dbhistory.database1"
    }
}
```

Run `docker-compose up -d` and connect to the Debezium container:

```
docker exec -it warpstream-debezium-1 /bin/bash
```

Create a Connector in Debezium:

```
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:8083/connectors -d @/kafka/connector.json
```

Check Connectors:

```
curl localhost:8083/connectors
```
