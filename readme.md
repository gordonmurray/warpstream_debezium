# CDC from MariaDB using Debezium and WarpStream

Trying out WarpStream as a Kafka alternative with Debezium.

### Insecure connection

Make an insecure connection from Debezium to [WarpStream using plain text](/insecure/)

### Secure connection

Make a secure connection from Debezium to [WarpStream Serverless - using SASL](./sasl/)

### Apicurio Schema registry

Connect Debezium to WarpStream using [Avro serialization and Apicurio schema registry](/insecure_schema_registry/)

### Sink data to S3 using Parquet format

Connect Debezium securely to WarpStream using Avro serialization and [sink the data to S3 in Parquet format](/secure_s3_sink)

---

In each folder, an example docker-compose.yml file is provided that launches a mariadb and a debezium container.

A connector can then submitted to Debezium which will tell it to read from Mariadb and push events to WarpStream.

![Debezium and WarpStream](./images/debezium_warpstream.png)

