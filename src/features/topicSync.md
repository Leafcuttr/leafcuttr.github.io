## Topic sync mode

By default, topic data is written to the operating system's page cache and flushed to the disk asynchronously. That improves throughput, but it also creates a short window where data could be lost if a crash or power failure occurs before the OS forces the pages to stable storage.

The common way to limit that risk is to increase replication and use `acks=all`, so that a single node failure doesn't cause data loss. Another (more conservative) option is to force a disk flush after every message (for example, `flush.interval.messages=1`), but that significantly reduces throughput and increases latency.

On resource-constrained or single-node deployments where replication isn't practical, LeafCuttr offers "topic sync mode." In this mode log segment files are opened with the O_DSYNC flag, which ensures data is committed to the physical medium before the producer is acknowledged. This approach provides stronger durability than the default behavior while avoiding the per-message fsync overhead of forcing a separate flush for every message.

Enable this mode globally or per-topic by setting `lc.log.sync.always` to `true`.

### Benefits
- Improved durability on single-node or edge deployments where replication isn't available
- Lower syscall overhead than forcing a flush after every message, so better throughput than per-message fsync
- Can be enabled globally or selectively for critical topics

### Caveats and limitations
- This affects only the log segment data; index and offset files are still flushed according to your `flush.interval.*` settings.
- Topic sync mode improves durability but is not a replacement for replication when high availability and redundancy are required.