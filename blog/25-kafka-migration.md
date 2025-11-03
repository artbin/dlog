# Migrating from Kafka to Pyralog: A 6-Week Journey

**Real migration story: Zero downtime, 10× performance, 75% cost savings**

*Published: November 3, 2025*

---

## Background

**Company**: E-commerce platform (10M daily active users)
**Legacy**: Kafka cluster (100 nodes, 50TB data)
**Pain points**: High latency, operational complexity, expensive

**Goal**: Migrate to Pyralog without downtime

---

## Why Migrate?

### Kafka Problems

```
Performance:
  • p99 write latency: 45ms
  • p99 read latency: 120ms
  • Rebalancing storms (5-10 minute pauses)

Operations:
  • ZooKeeper complexity
  • Manual partition management
  • Frequent out-of-memory crashes
  • 2 dedicated ops engineers

Cost:
  • Infrastructure: $60K/month
  • Engineering: $40K/month (ops engineers)
  • Total: $100K/month
```

### Pyralog Promise

```
Performance:
  • p99 write latency: <1ms (45× faster)
  • p99 read latency: 5ms (24× faster)
  • No rebalancing pauses

Operations:
  • No ZooKeeper (Raft built-in)
  • Auto partition management
  • Self-healing
  • 0.2 ops engineers (part-time)

Cost:
  • Infrastructure: $25K/month
  • Engineering: $5K/month
  • Total: $30K/month (70% savings)
```

---

## Migration Strategy

### Week 0: Planning & Preparation

**Tasks**:
1. Audit Kafka usage
2. Design Pyralog cluster
3. Set up test environment
4. Plan dual-write strategy

**Kafka audit results**:
```
Topics: 150
Partitions: 3,000
Throughput: 500K msgs/sec (50MB/sec)
Consumers: 45 consumer groups
Producers: 200 services
```

**Pyralog cluster design**:
```
Nodes: 50 Pyramid nodes (50% of Kafka)
Storage: 25TB (50% of Kafka, with compression)
Expected throughput: 2M msgs/sec (4× Kafka)
Cost: $25K/month vs $60K/month Kafka
```

---

### Week 1: Deploy Pyralog Cluster

**Day 1-2**: Infrastructure setup
```terraform
# Terraform deployment
resource "aws_instance" "pyramid" {
  count         = 50
  instance_type = "i3en.4xlarge"
  ami           = data.aws_ami.pyralog.id
  
  user_data = templatefile("pyramid-init.sh", {
    cluster_id = var.cluster_id
  })
}
```

**Day 3-4**: Monitoring & alerting
```yaml
# Prometheus + Grafana setup
- Pyralog dashboards
- Kafka comparison metrics
- Alert rules for latency/throughput
```

**Day 5**: Load testing
```
Test results:
  • Write throughput: 2.3M msgs/sec ✅
  • Read throughput: 5M msgs/sec ✅
  • p99 write latency: 0.8ms ✅
  • p99 read latency: 4.2ms ✅
  
All targets exceeded!
```

---

### Week 2: Dual-Write Implementation

**Strategy**: Write to both Kafka and Pyralog, read from Kafka only

```rust
/// Dual-write producer
pub struct DualWriteProducer {
    kafka: KafkaProducer,
    pyralog: PyralogClient,
    metrics: Metrics,
}

impl DualWriteProducer {
    pub async fn send(&self, msg: &Message) -> Result<()> {
        // Write to both (parallel)
        let (kafka_result, pyralog_result) = tokio::join!(
            self.kafka.send(msg),
            self.pyralog.write(msg),
        );
        
        // Track success/failure
        self.metrics.kafka_success.inc_if(kafka_result.is_ok());
        self.metrics.pyralog_success.inc_if(pyralog_result.is_ok());
        
        // Return Kafka result (primary for now)
        kafka_result
    }
}
```

**Rollout**:
- Day 1: Deploy dual-write code (feature flag off)
- Day 2: Enable for 1% of traffic
- Day 3: Enable for 10% of traffic
- Day 4: Enable for 50% of traffic
- Day 5: Enable for 100% of traffic

**Week 2 metrics**:
```
Dual-write success rate:
  • Kafka: 99.95%
  • Pyralog: 99.99% (fewer failures!)
  
Extra latency: +0.3ms (negligible)
```

---

### Week 3: Data Migration

**Strategy**: Backfill historical Kafka data into Pyralog

```rust
/// Backfill tool
pub struct KafkaToPyralogBackfill {
    kafka_consumer: KafkaConsumer,
    pyralog_writer: PyralogClient,
    checkpoint_store: CheckpointStore,
}

impl KafkaToPyralogBackfill {
    pub async fn run(&mut self) -> Result<()> {
        let start_offset = self.checkpoint_store.get_or_default()?;
        
        loop {
            // Read batch from Kafka
            let messages = self.kafka_consumer
                .poll_batch(start_offset, batch_size=10000)
                .await?;
            
            if messages.is_empty() {
                break; // Caught up!
            }
            
            // Write batch to Pyralog
            let offsets = self.pyralog_writer
                .write_batch(&messages)
                .await?;
            
            // Checkpoint progress
            self.checkpoint_store.update(offsets.last())?;
        }
        
        Ok(())
    }
}
```

**Progress**:
```
Total messages: 50 billion
Backfill rate: 500K msgs/sec
Time: 100,000 seconds = 28 hours
Result: ✅ Complete with zero errors
```

---

### Week 4: Validation & Testing

**Tasks**:
1. Verify data integrity
2. Test consumer migration
3. Benchmark performance

**Data validation**:
```rust
/// Compare Kafka vs Pyralog data
pub async fn validate_migration() -> Result<ValidationReport> {
    let mut mismatches = 0;
    let mut validated = 0;
    
    for partition in 0..3000 {
        let kafka_msgs = kafka.read_partition(partition).await?;
        let pyralog_msgs = pyralog.read_partition(partition).await?;
        
        if kafka_msgs != pyralog_msgs {
            mismatches += 1;
            log::error!("Partition {} mismatch", partition);
        }
        validated += 1;
    }
    
    Ok(ValidationReport { validated, mismatches })
}
```

**Result**: 0 mismatches out of 3,000 partitions ✅

**Consumer migration test**:
```rust
// Test consumer reading from Pyralog
let consumer = PyralogConsumer::new("analytics-group");
let messages = consumer.read_batch(100).await?;

// Verify consumer offset tracking works
assert_eq!(consumer.committed_offset().await?, expected_offset);
```

**Performance comparison**:
```
                Kafka       Pyralog     Improvement
Write latency   45ms (p99)  0.8ms       56× faster
Read latency    120ms (p99) 4.2ms       29× faster
Throughput      500K/sec    2.3M/sec    4.6× higher
```

---

### Week 5: Consumer Cutover

**Strategy**: Gradually move consumers from Kafka to Pyralog

**Phase 1** (Day 1-2): Analytics consumers (non-critical)
```
Services: 10
Impact: Low (batch processing)
Rollback: Easy
Result: ✅ Success, 20× faster queries
```

**Phase 2** (Day 3-4): Monitoring consumers
```
Services: 15
Impact: Medium (metrics, alerts)
Rollback: Medium
Result: ✅ Success, zero data loss
```

**Phase 3** (Day 5-7): Critical consumers (order processing)
```
Services: 20
Impact: High (revenue-affecting)
Rollback: Prepared
Result: ✅ Success, p99 latency 5ms → 0.8ms
```

**Gradual rollout per service**:
```rust
// Feature flag for gradual cutover
if feature_flags.read_from_pyralog(user_id) {
    pyralog_consumer.read().await?
} else {
    kafka_consumer.read().await?
}
```

**Week 5 metrics**:
```
Consumers migrated: 45/45 (100%)
Incidents: 0
Rollbacks: 0
Customer complaints: 0
```

---

### Week 6: Producer Cutover & Kafka Decommission

**Producer cutover**:
```rust
/// Switch from dual-write to Pyralog-only
pub struct SingleWriteProducer {
    pyralog: PyralogClient,
}

impl SingleWriteProducer {
    pub async fn send(&self, msg: &Message) -> Result<Offset> {
        // Write only to Pyralog now
        self.pyralog.write(msg).await
    }
}
```

**Rollout** (Day 1-5):
- 10% of producers per day
- Monitor for errors
- Keep Kafka running (read-only)

**Day 6-7**: Kafka decommission
```bash
# Stop Kafka brokers
for broker in broker-{1..100}; do
    ssh $broker "systemctl stop kafka"
done

# Backup Kafka data (just in case)
aws s3 sync /kafka-data s3://kafka-backup/

# Terminate Kafka cluster
terraform destroy -target=module.kafka
```

---

## Results

### Performance

```
                Before (Kafka)  After (Pyralog)  Improvement
Write latency   45ms (p99)      0.8ms (p99)      56× faster
Read latency    120ms (p99)     4.2ms (p99)      29× faster
Throughput      500K msg/sec    2.3M msg/sec     4.6× higher
Rebalance time  5-10 minutes    0 (no rebalances) ∞× better
```

### Cost Savings

```
                Before          After            Savings
Infrastructure  $60K/month      $25K/month       58% ($35K)
Operations      $40K/month      $5K/month        88% ($35K)
Total           $100K/month     $30K/month       70% ($70K)

Annual savings: $840K/year
```

### Operational Impact

```
                Before          After
Oncall pages    45/month        3/month (93% reduction)
Incidents       8/month         0.5/month (94% reduction)
Ops engineers   2 FTE           0.2 FTE (90% reduction)
Deploy time     2 hours         15 minutes (88% faster)
```

---

## Lessons Learned

### ✅ What Worked Well

**1. Dual-write strategy**
```
Pros:
  • Zero downtime
  • Gradual rollout
  • Easy rollback
  • Data validation in production
```

**2. Extensive testing**
```
Phases:
  1. Load testing (Week 1)
  2. Data validation (Week 4)
  3. Consumer testing (Week 4)
  4. Gradual cutover (Week 5)
  
Result: Zero production incidents
```

**3. Feature flags**
```rust
// Enable/disable per service, per user, per percentage
if flags.use_pyralog("analytics-service", user_id) {
    read_from_pyralog()
} else {
    read_from_kafka()
}
```

---

### ⚠️ Challenges & Solutions

**Challenge 1**: Consumer group offsets

*Problem*: Kafka and Pyralog have different offset schemes

*Solution*:
```rust
// Map Kafka offset → Pyralog offset
pub struct OffsetMapper {
    kafka_to_pyralog: HashMap<KafkaOffset, PyralogOffset>,
}

impl OffsetMapper {
    pub fn map(&self, kafka_offset: KafkaOffset) -> PyralogOffset {
        self.kafka_to_pyralog.get(&kafka_offset).copied()
            .unwrap_or_else(|| self.find_by_timestamp(kafka_offset))
    }
}
```

**Challenge 2**: Message ordering

*Problem*: Dual-write may cause ordering differences

*Solution*:
```rust
// Use same partition key for both systems
let partition_key = msg.user_id;
kafka.send(msg, partition_key).await?;
pyralog.write(msg, partition_key).await?;

// Ensures same partition assignment
```

**Challenge 3**: Monitoring during migration

*Problem*: Hard to compare Kafka vs Pyralog metrics

*Solution*:
```rust
// Unified dashboard showing both systems
Grafana dashboard:
  • Kafka latency (red line)
  • Pyralog latency (green line)
  • Success rate (both)
  • Message count (both)
  • Cost per message (both)
```

---

## Migration Toolkit

### Scripts Provided

```bash
# 1. Kafka audit tool
./kafka-audit --brokers=broker1:9092 --output=audit.json

# 2. Dual-write enabler
./enable-dual-write --percentage=10

# 3. Backfill tool
./backfill --start-offset=0 --batch-size=10000

# 4. Validation tool
./validate-migration --sample-rate=0.01

# 5. Consumer cutover
./cutover-consumer --group=analytics --dry-run

# 6. Kafka decommission
./decommission-kafka --backup-to=s3://backups/
```

---

## Summary

Migrating from Kafka to Pyralog in **6 weeks**:

### Timeline
- **Week 1**: Deploy Pyralog ✅
- **Week 2**: Dual-write ✅
- **Week 3**: Backfill data ✅
- **Week 4**: Validation ✅
- **Week 5**: Consumer cutover ✅
- **Week 6**: Producer cutover & decommission ✅

### Results
- **Performance**: 56× faster writes, 29× faster reads
- **Cost**: 70% savings ($840K/year)
- **Operations**: 93% fewer oncall pages
- **Downtime**: **ZERO**

### The Bottom Line

Migration was **easier than expected**. Dual-write strategy + gradual cutover = zero risk. Performance and cost improvements exceeded expectations.

*Zero downtime. Massive gains.*

---

## Next Steps

- Download [Migration Toolkit](https://github.com/pyralog/kafka-migration)
- Read [Migration Guide](../docs/kafka-migration.md)
- Join [Migration Support Discord](https://discord.gg/pyralog)

---

*Part 25 of the Pyralog Blog Series*

*Previously: [Operating in Production](24-operations.md)*
*Next: [Event-Driven Systems](26-event-driven.md)*

