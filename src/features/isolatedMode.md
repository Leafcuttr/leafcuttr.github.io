# Isolated Mode

When Isolated Mode is enabled, the broker operates in an optimised code path with replication disabled. This can give a performance boost and hence can be used in situations where it is not possible or desirable to have replication.

To enable this mode, set `lc.isolated` to `true`.

In isolated mode, you may want to use [Topic Sync](./topicSync.md) for critical topics.