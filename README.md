# Kafka + Spark Structured Streaming + SQL Database

This project provides a local development environment using **Docker Compose** to run:

* **Zookeeper**
* **Kafka Broker**
* **Spark (client or cluster)**
* **SQL Database** (SSMS)

The goal is to stream data from Kafka into Spark, process it, and write the results to a SQL database.

---

## Requirements

* Docker & Docker Compose installed
* Spark application that reads from Kafka and writes to SQL using JDBC

---

## How to Run the Environment

Run the full environment:

```bash
docker compose up -d
```

Make sure you:

1. Configure the **connection between Spark and the SQL database**
2. Configure the **Kafka advertised listener** so Spark can connect
3. Configure the **Spark host** to ensure proper communication

All details are explained below.

---

#  Configuration Details

##  Configure Kafka Host

Inside `docker-compose.yml`, set:

```yaml
KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
```

Or replace `localhost` with your machine IP if Spark is running outside Docker.

Spark must use the same address:

```python
kafkaServer = "localhost:9092"
```

---

##  Configure SQL Database Connection

Your Spark application must include a valid JDBC configuration:

```python
db_url = "jdbc:sqlserver://localhost:1433;databaseName=(Database_name)"
db_user = "(User's_name)"
db_pass = "(User's_Password)"

properties = {
    "user": db_user,
    "password": db_pass,
    "driver": "com.microsoft.sqlserver.jdbc.SQLServerDriver"
}
```
Make sure your Spark container or host machine has access to the database port.
---

##  Configure Spark Host

If Spark is running **inside Docker**, use service names to connect:

```python
kafkaServer = "kafka:9092"
sqlHost = "sqlserver"   # example
```

If Spark is running **on your host machine**, use:

```python
kafkaServer = "localhost:9092"
sqlHost = "localhost"
```

---

#  Example: Spark Structured Streaming Code

```python
raw_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", kafkaServer) \
    .option("subscribe", "my-topic") \
    .load()

# Write to SQL
query = processed_df.writeStream \
    .foreachBatch(lambda df, epochId: df.write.jdbc(db_url, "output_table", "append", properties)) \
    .outputMode("append") \
    .start()

query.awaitTermination()
```

---

#  Example docker-compose.yml (simplified)

```yaml
version: '3.8'

services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      SA_PASSWORD: "YourStrongPassword"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
```
# Then for the dashboards and visualization use node-red locally by downloading it and connecting it with the flows in
the folder then operating all the four source codes together
---

# Summary

To successfully run the project, **you must correctly configure**:

* The **JDBC connection** between Spark and SQL database
* The **Kafka advertised listener** (host/IP)
* The **Spark host** so it can reach Kafka & SQL

"Thank YOU !!!!!!!!! XD"
