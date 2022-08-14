# InfluxDB cookbook


## Shell

### Open a CLI on server host

```bash
influx -precision rfc3339 -database mydb
```

### Execute an influxQL command

```bash
influx -database mydb -execute "SELECT * FROM my_ts LIMIT 10"
```

```bash
influx -execute 'SELECT * FROM "mydb"..my_ts LIMIT 10'
```

### Export DB into InfluxDB line protocol format (readable)

```bash
sudo influx_inspect export -datadir /var/lib/influxdb/data/ -waldir /var/lib/influxdb/wal/ -database mydb -lponly -out mydb_export.txt
```

### Import DB from InfluxDB line protocol format

```bash
curl -i -XPOST "http://localhost:8086/write?db=mydb" --data-binary @mydb_export.txt
```


## InfluxQL commands

### Copy a measurement to another 

**Add GROUP BY for avoid tags become fields on destination.**

```sql
SELECT * INTO ts_dest FROM ts_source GROUP BY *
```

### Show tags/fields for a measurement

```sql
SHOW TAG KEYS FROM ts
```

```sql
SHOW FIELD KEYS FROM ts
```

### Delete a time series (also call measurement)

```sql
DROP MEASUREMENT ts_to_remove
```

### Update default retention policy to 120 days with shard of 7 days

```sql
ALTER RETENTION POLICY autogen ON mydb DURATION 120d SHARD DURATION 7d DEFAULT
```
