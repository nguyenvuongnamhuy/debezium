1. Start services
  docker compose -f docker-compose-newblink.yml up -d
  docker restart debezium <!-- restart debezium to re-connect kafka -->

2. Set wal_level = logical
  docker exec -it postgres bash
  apt-get update && apt-get install nano
  nano /var/lib/postgresql/data/postgresql.conf
  input: `wal_level = logical`
  docker restart postgres

3. set replica identity
  create table "user" (id integer primary key, email varchar);
  alter table public."user" replica identity full;

4. create connector
  docker cp ./debezium.json debezium:/debezium.json
  docker exec -i debezium curl -H 'Content-Type: application/json' 0.0.0.0:8083/connectors --data @/debezium-blink.json
  <!-- docker exec -i debezium curl --location 'http://localhost:8083/connectors' \
    --header 'Accept: application/json' \
    --header 'Content-Type: application/json' \
    --data @/debezium-blink.json -->
## connector

```bash
{
  "name": "blink-cdc-using-debezium-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "plugin.name": "pgoutput",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "123456",
    "database.dbname": "mybl-workspace",
    "database.server.name": "mybl-workspace",
    "table.include.list": "public.TB_ACCOUNT, public.TB_ANS_TYPE, public.TB_ANSWER",
    "topic.prefix": "mybl-workspace"
  }
}
```

### topic

```bash
`${topic.prefix}.${table.include.list}
```

---

## list debezium connectors

docker exec -i debezium curl http://localhost:8083/connectors

## remove debezium connector

docker exec -i debezium curl -X DELETE http://localhost:8083/connectors/cdc-using-debezium-connector

## consume kafka message

docker exec kafka /opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic cdc-using-debezium-topic

## describe topic

docker exec -it kafka bash
cd opt/bitnami/kafka/bin/
./kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic cdc-using-debezium-topic

## replace local ip

ifconfig | grep inet
