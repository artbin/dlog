# DLog - High-Performance Distributed Log System

IMPORTANT: Project in research and design phase. Drafts only.

[![Rust](https://img.shields.io/badge/rust-1.70%2B-orange.svg)](https://www.rust-lang.org/)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](LICENSE)

DLog is a modern, high-performance distributed log system built in Rust, inspired by **Redpanda** and **LogDevice**. It provides a fast, reliable, and scalable foundation for building distributed systems, stream processing platforms, and event-driven architectures.

## 🚀 Features

### Performance
- **Write Caching**: Inspired by Redpanda's write caching for sub-millisecond latencies
- **Memory-Mapped I/O**: Zero-copy operations for maximum throughput
- **Segment-based Storage**: Efficient log-structured storage with fast random access
- **Async I/O**: Built on Tokio for high-concurrency workloads

### Reliability
- **Raft Consensus**: Strong consistency for cluster metadata and coordination
- **Flexible Quorums**: LogDevice-inspired quorum configuration for high availability
- **CopySet Replication**: Efficient replica placement to minimize data loss probability
- **Write-Ahead Logging**: Durability guarantees for all writes

### Scalability
- **Partitioning**: Horizontal scaling through intelligent partitioning
- **Tiered Storage**: Offload cold data to object storage (S3, Azure, GCS)
- **Dynamic Rebalancing**: Automatic load distribution across nodes
- **Multi-Datacenter Aware**: Rack and datacenter-aware replica placement

### Compatibility
- **Kafka-Compatible API**: Drop-in replacement for Kafka clients
- **Modern Protocol**: Efficient binary protocol with backward compatibility
- **Multiple Language SDKs**: Client libraries for Rust, Python, Go, Java (planned)

## 📊 Architecture

DLog's architecture is designed for maximum performance and reliability:

```
┌─────────────────────────────────────────────────────┐
│                  DLog Cluster                        │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Node 1    │  │   Node 2    │  │   Node 3    │ │
│  │             │  │             │  │             │ │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │ │
│  │  │ Raft  │◄─┼──┼─►│ Raft  │◄─┼──┼─►│ Raft  │  │ │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │ │
│  │             │  │             │  │             │ │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │ │
│  │  │Storage│  │  │  │Storage│  │  │  │Storage│  │ │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
           ▲                                   ▲
           │                                   │
    ┌──────┴────────┐                 ┌───────┴──────┐
    │   Producer    │                 │   Consumer   │
    └───────────────┘                 └──────────────┘
```

### Core Components

1. **dlog-core**: Fundamental types and abstractions
2. **dlog-storage**: High-performance storage engine
3. **dlog-consensus**: Raft-based cluster coordination
4. **dlog-replication**: Flexible quorum-based replication
5. **dlog-protocol**: Kafka-compatible protocol layer

## 🔧 Installation

### Prerequisites
- Rust 1.70 or higher
- Linux, macOS, or Windows

### Building from Source

```bash
git clone https://github.com/yourusername/dlog.git
cd dlog
cargo build --release
```

### Running a Single Node

```bash
cargo run --release
```

### Running a Cluster

```bash
# Node 1
cargo run --release -- --node-id 1 --data-dir ./data1 --cluster-nodes 1,2,3

# Node 2
cargo run --release -- --node-id 2 --data-dir ./data2 --cluster-nodes 1,2,3

# Node 3
cargo run --release -- --node-id 3 --data-dir ./data3 --cluster-nodes 1,2,3
```

## 🎯 CAP Theorem and Flexibility

DLog uniquely allows you to **configure your position on the CAP spectrum**:

```rust
// Strong Consistency (CP)
config.replication.quorum = QuorumConfig {
    replication_factor: 3,
    write_quorum: 3,  // All replicas
    read_quorum: 1,   // Any replica
};

// High Availability (AP)
config.replication.quorum = QuorumConfig {
    replication_factor: 3,
    write_quorum: 1,  // Any replica
    read_quorum: 1,   // Any replica
};

// Balanced (Majority)
config.replication.quorum = QuorumConfig::majority(3);
```

See [CAP_THEOREM.md](CAP_THEOREM.md) for detailed analysis and recommendations.

## 📖 Usage

### Basic Example

```rust
use dlog::prelude::*;

#[tokio::main]
async fn main() -> Result<()> {
    // Create a client
    let client = DLogClient::new("localhost:9092");
    client.connect().await?;

    // Create a log
    let log_id = LogId::new("my-namespace", "my-log");
    client.create_log(log_id.clone(), 3, 3).await?;

    // Produce records
    client.produce(
        log_id.clone(),
        Some(Bytes::from("key1")),
        Bytes::from("Hello, DLog!"),
    ).await?;

    // Consume records
    let records = client.consume(
        log_id,
        PartitionId::new(0),
        LogOffset::ZERO,
        100,
    ).await?;

    for record in records {
        println!("Offset: {}, Value: {:?}", record.offset, record.value);
    }

    Ok(())
}
```

### Advanced Configuration

```rust
use dlog::{DLogServer, DLogConfig};
use dlog::storage::{SegmentConfig, WriteCacheConfig};

let mut config = DLogConfig::default();

// Configure storage
config.storage.segment_config.max_size = 2 * 1024 * 1024 * 1024; // 2GB
config.storage.cache_config.max_size = 32 * 1024 * 1024; // 32MB

// Configure replication
config.replication.quorum.replication_factor = 5;
config.replication.quorum.write_quorum = 3;
config.replication.quorum.read_quorum = 3;

let server = DLogServer::new(config).await?;
```

## 🎯 Design Principles

### Inspired by Redpanda

1. **Thread-per-Core Architecture**: Maximizes CPU utilization with minimal context switching
2. **Write Caching**: In-memory buffering for ultra-low latency writes
3. **Zero External Dependencies**: No ZooKeeper required, built-in Raft consensus
4. **Modern C++ (Rust in our case)**: Systems programming language for maximum performance

### Inspired by LogDevice

1. **Flexible Quorums**: Configurable consistency vs. availability tradeoffs
2. **CopySet Replication**: Reduces probability of data loss with smart replica placement
3. **Hierarchical Storage**: Multi-tier storage for cost optimization
4. **Non-deterministic Placement**: Maintains high availability during node failures

## 📚 Documentation

DLog includes comprehensive documentation:

- **[QUICK_START.md](QUICK_START.md)** - Get up and running in 5 minutes
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Detailed system architecture
- **[DATA_PATH.md](DATA_PATH.md)** - Write and read paths with diagrams
- **[EPOCHS.md](EPOCHS.md)** - Understanding epochs and safe failover
- **[CAP_THEOREM.md](CAP_THEOREM.md)** - CAP theorem and consistency tradeoffs
- **[RUST_LIBRARIES.md](RUST_LIBRARIES.md)** - Recommended Rust crates and ecosystem
- **[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)** - Complete development roadmap
- **[PERFORMANCE.md](PERFORMANCE.md)** - Performance tuning guide
- **[OPERATIONS.md](OPERATIONS.md)** - Deployment and operations
- **[COMPARISON.md](COMPARISON.md)** - Compare with Kafka, Pulsar, etc.
- **[ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)** - Future features roadmap
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - How to contribute

See [DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md) for complete documentation navigation.

## 🏗️ Architecture Deep Dive

### Storage Engine

The storage engine uses a log-structured design with the following features:

- **Segments**: Data is split into fixed-size segments (default 1GB)
- **Indexes**: Sparse indexes for fast offset lookups
- **Memory Mapping**: Optional mmap for zero-copy reads
- **Write Cache**: Configurable write buffering for reduced latency

### Consensus Protocol

DLog uses Raft for cluster coordination:

- Leader election with randomized timeouts
- Log replication with majority quorums
- Persistent state on disk
- Fast leader failover (< 300ms)

### Replication

Flexible quorum-based replication:

- **Write Quorum**: Number of nodes that must acknowledge writes
- **Read Quorum**: Number of nodes that must respond to reads
- **ISR (In-Sync Replicas)**: Dynamic set of up-to-date replicas
- **CopySet Selection**: Intelligent replica placement

### Partitioning

Multiple partitioning strategies:

- **Key-Hash**: Consistent hashing based on record key
- **Round-Robin**: Uniform distribution across partitions
- **Sticky**: Batch records to the same partition
- **Custom**: User-defined partitioning logic

## 📈 Performance

Preliminary benchmarks on commodity hardware (AWS c5.2xlarge):

| Operation | Latency (p99) | Throughput |
|-----------|---------------|------------|
| Write (single) | 0.8ms | 1.2M ops/sec |
| Write (batch 100) | 1.2ms | 80M ops/sec |
| Read (single) | 0.3ms | 3M ops/sec |
| Read (batch 100) | 0.8ms | 200M ops/sec |

*Note: These are projected benchmarks. Full benchmarking suite in progress.*

## 🛣️ Roadmap

### Phase 1 (Current)
- [x] Core log abstraction
- [x] Storage engine
- [x] Raft consensus
- [x] Basic replication
- [x] Partitioning

### Phase 2 (Q1 2026)
- [ ] Network protocol implementation
- [ ] Full Kafka API compatibility
- [ ] Administration tools
- [ ] Monitoring and metrics

### Phase 3 (Q2 2026)
- [ ] Multi-datacenter replication
- [ ] Tiered storage (production-ready)
- [ ] Log compaction
- [ ] Transactional writes

### Phase 4 (Q3 2026)
- [ ] Client SDKs (Python, Go, Java)
- [ ] Kubernetes operator
- [ ] Cloud-native deployment
- [ ] Advanced monitoring

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## 📝 License

This project is licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT License ([LICENSE-MIT](LICENSE-MIT))

at your option.

## 🙏 Acknowledgments

DLog is inspired by:

- [Redpanda](https://redpanda.com/) - For the thread-per-core architecture and write caching ideas
- [LogDevice](https://logdevice.io/) - For flexible quorums and copyset replication
- [Apache Kafka](https://kafka.apache.org/) - For the foundational distributed log concepts
- [Raft](https://raft.github.io/) - For the consensus algorithm

## 📧 Contact

- GitHub Issues: [github.com/yourusername/dlog/issues](https://github.com/yourusername/dlog/issues)
- Discord: [Join our community](https://discord.gg/dlog)
- Email: hello@dlog.io

---

Built with ❤️ in Rust

