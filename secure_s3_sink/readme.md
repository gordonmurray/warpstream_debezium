
# CDC from MariaDB using Debezium and WarpStream - with Avro serialization and Apicurio Schema Registry - Sinking to S3 in Parquet format

This example shows using Debezium to capture data from a MariaDB and also using Debezium to sink the resulting data to S3 in Parquet format.

## What is Apache Parquet

Apache Parquet is a columnar storage format available to any project in the Hadoop ecosystem, regardless of the choice of data processing framework, data model or programming language. [[Read more](https://parquet.apache.org/)]

## What is Avro serialization?

Apache Avro is a compact, efficient binary format that provides rich data structures and a fast, binary data serialization and deserialization framework. [[Read more](https://avro.apache.org/)]

## What is an Apicurio Schema Registry?

Apicurio schema registry stores Avro schemas and provides a central repository for version-controlled schemas, enhancing data governance. [[Read more](https://www.apicur.io/registry/)]

## Why use Avro and Apicurio schema registry?

Avro serializtion compacts the data for effective transfer and storage. Apicurio stores the schemas so the data can be serialized and deserialized  correctly.

Debezium captures database changes (inserts, updates, deletes) and publishes them to Kafka topics in Avro format. Apicurio Schema Registry stores the Avro schemas used by Debezium, allowing consumers to understand the data structure. Kafka consumers like Apache Flink retrieve the data and the schema from the schema registry, ensuring that the data is serialized correctly.

## Kafka Connect S3

This example uses the Amazon S3 Sink Connector which can be downloaded from https://www.confluent.io/hub/confluentinc/kafka-connect-s3/

## Usage

Use Docker Compose to start Mariadb, Debezium and Apicurio containers to try to out CDC. The database will be preloaded with a products table with some sample products.

A Debezium connector will snapshot the data and capture ongoing changes and send the data to WarpStream. The schemas of the tables will be stored in Apicurio.

A second connector will read the topic from WarpStream and send the data to S3 and store the data in Parquet format.

Minio is used to simulate S3.

To get started, update the docker-compose.yml file and the files/connector_source.json with the cluster credentials provided by WarpStream.

Start the containers:

```
docker-compose up -d
```

Connect to the Debezium container:


```
docker exec -it debezium /bin/bash
```

Create a source Mariadb sonnector:

```
curl -X PUT http://localhost:8083/connectors/my_database/config -H "Content-Type: application/json" -d @connector_source.json
```

Create an S3 sink connector:

```
curl -X PUT http://localhost:8083/connectors/s3/config -H "Content-Type: application/json" -d @connector_sink.json
```

List Connectors:

```
curl localhost:8083/connectors
```

Check Connector status:

```
curl localhost:8083/connectors/my_database/status
curl localhost:8083/connectors/s3/status
```

The UI for Apicurio will be available at http://localhost:8081

You'll see some schemas created in the UI, such as `testing.mydatabase.products-value`

![Apicurio UI](files/apicurio.png)

### Check Minio for resulting files

Connect to the Minio container and view the contents of the "mybucket" bucket to see the resulting Parquet data

```
docker exec -it minio /bin/bash

ls -h /data/mybucket/topics/testing.mydatabase.products/
````

You'll see some data in the bucket along the lines of of the following with the folders named with combination of the prefix, database and table names:

```
data
└── mybucket
    └── topics
        └── testing.mydatabase.products
            └── partition=0
                ├── testing.mydatabase.products+0+0000000003.snappy.parquet
                │   ├── 1fb3362e-fd1a-49fc-81d0-dff33b1cd5bc
                │   │   └── part.1
                │   └── xl.meta
                ├── testing.mydatabase.products+0+0000000006.snappy.parquet
                │   ├── 55d5a657-a562-44b5-90f7-ae0fc40eee75
                │   │   └── part.1
                │   └── xl.meta
                ├── testing.mydatabase.products+0+0000000009.snappy.parquet
                │   ├── b4f8e214-ef69-4af6-abc5-27d6e50aa6d9
                │   │   └── part.1
                │   └── xl.meta
                ├── testing.mydatabase.products+0+0000000012.snappy.parquet
                │   ├── d5ebc093-95e7-4e01-acba-b8112de0950b
                │   │   └── part.1
                │   └── xl.meta
```