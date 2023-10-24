
```bash
docker exec -it sql-client /bin/bash
```

./sql-client.sh

```bash
Flink SQL> show catalogs;
default_catalog

Flink SQL> show databases;
default_database
```

```bash
docker-compose exec broker bash
```

Now let's get a list of all topics currently on the broker: 

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

Confirms that we have a `user_behavior` topic configured; this means the Datagen connector is doing some work for us.  Let's go over to Flink now:

```bash
docker exec -it sql-client /bin/bash
```

No tables?

```
Flink SQL> show tables;
[INFO] Result was empty.
```

Ok - let's create one!

```sql
CREATE TABLE user_behavior (
    user_id BIGINT,
    item_id BIGINT,
    category_id BIGINT,
    behavior STRING,
    ts TIMESTAMP(3),
    proctime AS PROCTIME(),   -- generates processing-time attribute using computed column
    WATERMARK FOR ts AS ts - INTERVAL '5' SECOND  -- defines watermark on ts column, marks ts as event-time attribute
) WITH (
    'connector' = 'kafka',  -- using kafka connector
    'topic' = 'user_behavior',  -- kafka topic
    'scan.startup.mode' = 'earliest-offset',  -- reading from the beginning
    'properties.bootstrap.servers' = 'kafka:9094',  -- kafka broker address
    'format' = 'json'  -- the data format is json
);
```


You should see:

```
[INFO] Table has been created.
```

Let's confirm that the table is available:

```
Flink SQL> show tables;
user_behavior
```

Flink SQL> SELECT * FROM user_behavior;

Yikes - this needs to be fixed:

```log
[ERROR] Could not execute SQL statement. Reason:
org.apache.flink.runtime.rest.util.RestClientException: [org.apache.flink.runtime.rest.handler.RestHandlerException: Failed to deserialize JobGraph.
[...]
Caused by: java.lang.ClassCastException: cannot assign instance of java.util.ArrayList to field org.apache.flink.runtime.jobgraph.JobVertex.results of type java.util.Map in instance of org.apache.flink.runtime.jobgraph.JobVertex
```

This is the next thing that we need to fix.

Sources:

- https://flink.apache.org/2020/07/28/flink-sql-demo-building-an-end-to-end-streaming-application/
- https://github.com/wuchong/flink-sql-demo/blob/v1.11-EN/docker-compose.yml