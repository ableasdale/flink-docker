

```bash
FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager"
docker network create flink-network
```

```
docker run \
    --rm \
    --name=jobmanager \
    --network flink-network \
    --publish 8081:8081 \
    --env FLINK_PROPERTIES="${FLINK_PROPERTIES}" \
    flink:1.17.1-scala_2.12 jobmanager
```

```
docker run \
    --rm \
    --name=taskmanager \
    --network flink-network \
    --env FLINK_PROPERTIES="${FLINK_PROPERTIES}" \
    flink:1.17.1-scala_2.12 taskmanager
```


cd /Users/ableasdale/Documents/workspace/flink-1.17.1

./bin/flink run ./examples/streaming/TopSpeedWindowing.jar

(fails)