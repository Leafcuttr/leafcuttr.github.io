# Forward

Forwarding topics to another Kafka cluster.

This feature lets LeafCuttr replicate or forward topic data to a remote Kafka cluster. It's useful for backups, migrations, or bridging edge deployments to central clusters. Forwarding can be configured per-topic and supports preserving message ordering and metadata where possible.