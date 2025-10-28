# Topic Forwarding (WIP)

This feature allow forwarding local topics to external Kafka cluster.

This feature lets LeafCuttr replicate or forward topic data to a remote Kafka cluster. It's useful for backups, migrations, or bridging edge deployments to central clusters.

This currently uses an in-process MirrorMaker2 standalone instance under the hood.

Specify the path to the MM2 config file with the `lc.topic.forward.config` property.