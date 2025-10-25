
# MQTT Broker and Proxy

The MQTT Broker and Proxy is an embedded MQTT Broker in LeafCuttr that can also store messages from specified MQTT topics to Kafka topics.

- All configuration properties follow the standard Kafka configuration system and can be specified in the server properties file.
- Authentication is optional; if username and password are not set, clients can connect without credentials.

## Configuration

### Enable/Disable Configuration

**`lc.mqtt.broker.enable`**
- **Type:** Boolean
- **Default:** `true`
- **Description:** Controls whether the embedded MQTT broker is enabled. When set to `false`, the MQTT proxy will not start.

### MQTT Broker Settings

**`mqtt.host`**
- **Type:** String
- **Default:** `0.0.0.0`
- **Description:** The host address the embedded MQTT broker will bind to. The default value `0.0.0.0` means the broker will listen on all available network interfaces.

**`mqtt.port`**
- **Type:** Integer
- **Default:** `1883`
- **Range:** 1 to 65535
- **Description:** The port number for the embedded MQTT broker.

### Authentication Settings

**`mqtt.auth.username`**
- **Type:** String
- **Default:** `null` (no authentication required)
- **Description:** A shared username that all MQTT clients must use to connect. If not specified, authentication is disabled.

**`mqtt.auth.password`**
- **Type:** Password
- **Default:** `null` (no authentication required)
- **Description:** A shared password that all MQTT clients must use to connect. If not specified, authentication is disabled.

### Kafka Connection Settings

**`mqtt.kafka.bootstrap.servers`**
- **Type:** String
- **Default:** Uses the Kafka server's advertised brokers
- **Description:** A comma-separated list of host:port pairs to establish the initial connection to the Kafka cluster. If not specified, the proxy will use the Kafka server's advertised brokers.

**`mqtt.kafka.client.id`**
- **Type:** String
- **Default:** `mqtt-kafka-proxy`
- **Description:** An identifier for the Kafka producer client.

### Topic Mapping Configuration

**`mqtt.topic.mapping.<mqtt-topic-filter>=<kafka-topic>`**
- **Format:** Configuration properties with prefix `mqtt.topic.mapping.`
- **Description:** Defines how MQTT topics are mapped to Kafka topics. The portion after the prefix is the MQTT topic filter, and the value is the target Kafka topic name.
- **Wildcard Support:** The single-level wildcard `+` is supported in MQTT topic filters to match any value at a single topic level.
- **Example:** `mqtt.topic.mapping.devices/+/temperature=kafka-temperature-topic` maps any MQTT topic matching the pattern `devices/+/temperature` (e.g., `devices/sensor1/temperature`, `devices/sensor2/temperature`) to the Kafka topic `kafka-temperature-topic`.

The `+` wildcard matches exactly one topic level and can be used at any position in the topic filter.
