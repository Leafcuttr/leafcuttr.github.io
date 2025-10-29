# Getting Started

## Demo

See https://github.com/Leafcuttr/lc-demo

## Using Docker

Here's a sample Docker Compose file:

```yaml
version: '3.8'

volumes:
  lc-data:

services:
  kafka:
    image: ghcr.io/leafcuttr/kafkalite:lc-0.5.2
    volumes:
      - lc-data:/tmp/kafka-logs/
    network_mode: "host"
    environment:
      # Set these to true as required
      # KAFKA_LC_MQTT_BROKER_ENABLE: "false"
      # KAFKA_LC_SCHEMA_REGISTRY_ENABLE: "false"
      # KAFKA_LC_HTTP_PROXY_ENABLE: "false"
      # KAFKA_LC_LOG_SYNC_ALWAYS: "true"
      # KAFKA_LC_ISOLATED: "true"
      # KAFKA_LC_TOPIC_FORWARD_CONFIG: filePath  # make sure filePath is mounted into the container

      # Standard configuration for Kafka
      KAFKA_LISTENERS: CONTROLLER://localhost:9091,HOST://0.0.0.0:9092,DOCKER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: HOST://localhost:9092,DOCKER://kafka:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,DOCKER:PLAINTEXT,HOST:PLAINTEXT

      # Settings required for KRaft mode
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9091

      # Listener to use for broker-to-broker communication
      KAFKA_INTER_BROKER_LISTENER_NAME: DOCKER

      # Required for a single node cluster
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

## Quick Reference

* `lc.http.proxy.enable = true` to enable [HTTP Proxy](/features/httpProxy.md)
* `lc.mqtt.broker.enable = true` to enable[MQTT Broker](/features/mqttBrokerProxy.md)
* `lc.schema.registry.enable = true` to enable [Schema Registry](/features/schemaRegistry.md)
* `lc.log.sync.always = true` to enable [Topic Sync](/features/topicSync.md)
* `lc.isolated = true` to enable [Isolated Mode](/features/isolatedMode.md)

See the individual modules for details.

### Embedded Server mode
WIP