# CDC from MariaDB using Debezium and WarpStream

Trying out WarpStream as a Kafka alternative with Debezium.

### Secure

Connect Debezium to [WarpStream Serverless - using SASL](./sasl/)

### Insecure

Connect Debezium to [WarpStream using plain text](/insecure/)

### Apicurio Schema registry

Connect Debezium to WarpStream using [Avro serialization and Apicurio schema registry](/insecure_schema_registry/)

---

In each folder, an example docker-compose.yml file is provided that launches a mariadb and a debezium container.

A connector can then submitted to Debezium which will tell it to read from Mariadb and push events to WarpStream.

![Debezium and WarpStream](./images/debezium_warpstream.png)

