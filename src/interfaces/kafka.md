# Kafka

## example producer

```sh
docker run --net=host --rm -it ghcr.io/leafcuttr/kafkalite:lc-0.5.1 \
 /opt/kafka/bin/kafka-producer-perf-test.sh \
   --topic perfTest --num-records 10000000 \
   --throughput -1 --record-size 1000 \
   --producer-props bootstrap.servers=localhost:9092
```

## example consumer

```sh
docker run --net=host --rm -it ghcr.io/leafcuttr/kafkalite:lc-0.5.1 \
 /opt/kafka/bin/kafka-consumer-perf-test.sh \
  --topic perfTest --messages 10000000 --timeout 100000 \
  --bootstrap-server localhost:9092 --show-detailed-stats
```
