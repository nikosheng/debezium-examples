# Debezium KStreams - WebSockets Example

This demo shows how to use KStreams to join two CDC event streams created by Debezium,
do some calculation on the joined stream and push the merged events to a client using WebSockets.

The domain is that of weather stations that measure temperature data.
There's an application _event-source_, which persists random temperature measurements.
In reality this could for instance expose a REST API, to which individual sensors post their events.

The application has two tables:

* `stations`: Named weather stations
* `temperature_measurements`: temperature measurements for a station with a value and timestamp

Debezium is used to capture changes to the two tables in the application's underlying MySQL database.

The _temperature-aggregator_ application runs KStreams to join measurements with stations,
group the events by station name and calculate the average temperature per station.

The aggregated values are pushed to WebSockets.
For that purpose, the aggregator application exposes a WebSockets endpoint using WildFly Swarm.

## Preparations

Build data generator application and aggregator application:

```shell
mvn clean install -f event-source/pom.xml
mvn clean install -f temperature-aggregator/pom.xml
```

Start Kafka, Kafka Connect, MySQL, event source and aggregator:

```shell
export DEBEZIUM_VERSION=0.8
docker-compose up --build
```

Once you see the message "Waiting for source connector to be deployed" in the logs,
deploy the Debezium MySQL connector:

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @mysql-source.json
```

# Consume aggregated messages

Once you see the message "WildFly Swarm is Ready" in the logs, open the UI in the browser: http://localhost:8079/.
It shows a chart with the average temperature values which is updated in near-realtime as new temperature measurements arrive via the event generator app.

Alternatively, browse the Kafka topic:

```shell
docker-compose exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic average_temperatures_by_station
```

# Shut down the cluster

```shell
docker-compose down
```