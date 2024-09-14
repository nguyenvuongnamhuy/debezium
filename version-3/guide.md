1. Start services

```bash
docker compose up -d
```

2. Set wal_level = logical

Access into postgres container

```bash
docker exec -it debezium-postgres bash
```

Insert `wal_level = logical` directly

```bash
echo "wal_level = logical" >> /var/lib/postgresql/data/postgresql.conf
```

Or

```bash
apt-get update && apt-get install nano
nano /var/lib/postgresql/data/postgresql.conf
input: `wal_level = logical`
```

Restart postgres container

```bash
docker restart debezium-postgres
```

<!-- 3. set replica identity
create table "user" (id integer primary key, email varchar);
alter table public."user" replica identity full; -->

4. create connector

Copy connector config into debezium container

```bash
docker cp ./debezium.json debezium-debezium:/debezium.json
```

Register a new connector

```bash
docker exec -i debezium-debezium curl -H 'Content-Type: application/json' 0.0.0.0:8083/connectors --data @/debezium.json
```

---

## Show all connectors

docker exec -i debezium-debezium curl http://localhost:8083/connectors

## Remove a connector

docker exec -i debezium-debezium curl -X DELETE http://localhost:8083/connectors/debezium-connector
