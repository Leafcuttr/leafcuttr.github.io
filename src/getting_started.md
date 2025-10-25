# Getting Started

The simplest way of deploying Leafcuttr is with Docker:

```
docker run {imageName:version}
```

The Leafcuttr image provides the same commands and follows the same configuration conventions as the Apache Kafka image.

## Quick Reference

* `lc.http.proxy.enable = true` to enable [HTTP Proxy](/features/httpProxy.md)
* `lc.mqtt.broker.enable = true` to enable[MQTT Broker](/features/mqttBrokerProxy.md)
* `lc.schema.registry.enable = true` to enable [Schema Registry](/features/schemaRegistry.md)

See the individual modules for details.

### Embedded Server mode
WIP