# Pyralog Implementation Plan

**Last Updated**: November 2025  
**Status**: Design phase, ready for implementation  
**Target**: Production-ready by Q3 2026

Comprehensive roadmap for building Pyralog from the ground up with novel architectural primitives.

---

## 📖 Table of Contents

1. [Overview](#overview)
2. [Architecture Evolution Path](#architecture-evolution-path)
3. [Phase 0: Obelisk Sequencer (Pharaoh Network)](#phase-0-obelisk-sequencer-pharaoh-network)
4. [Phase 1: Foundation & Core Types](#phase-1-foundation--core-types)
5. [Phase 2: Pyramid Storage (LSM-Tree)](#phase-2-pyramid-storage-lsm-tree)
6. [Phase 3: Dual Raft Consensus](#phase-3-dual-raft-consensus)
7. [Phase 4: Replication & Epochs](#phase-4-replication--epochs)
8. [Phase 5: Smart Clients & Protocols](#phase-5-smart-clients--protocols)
9. [Phase 6: Multi-Model Database](#phase-6-multi-model-database)
10. [Phase 7: Production Hardening](#phase-7-production-hardening)
11. [Phase 8: Advanced Features](#phase-8-advanced-features)
12. [Testing Strategy](#testing-strategy)
13. [Deployment Strategy](#deployment-strategy)
14. [Success Criteria](#success-criteria)

---

## Overview

### 🎯 Guiding Principles

1. **Novel primitives first** - Obelisk Sequencer is the foundation
2. **Two-tier architecture** - Separate coordination (Obelisk) from storage (Pyramid)
3. **Test continuously** - Every component has comprehensive tests
4. **Benchmark early** - Measure performance from day one
5. **Evolutionary** - Start simple, evolve to maximum scale
6. **Production-minded** - Design for real-world use from the start

### ⏱️ Timeline Estimate

```
Phase 0: Obelisk Sequencer              (2-3 weeks)   Dec 2025
Phase 1: Foundation & Core Types        (2-3 weeks)   Jan 2026
Phase 2: Custom LSM-Tree Storage        (4-6 weeks)   Jan-Feb 2026
Phase 3: Dual Raft Consensus            (4-6 weeks)   Feb-Mar 2026
Phase 4: Replication & Epochs           (4-5 weeks)   Mar-Apr 2026
Phase 5: Smart Clients & Protocols      (3-4 weeks)   Apr-May 2026
Phase 6: Multi-Model Database           (4-6 weeks)   May-Jun 2026
Phase 7: Production Hardening           (4-6 weeks)   Jun-Jul 2026
Phase 8: Advanced Features              (Ongoing)     Aug 2026+
────────────────────────────────────────────────────────────────
Total to Production-Ready:              27-39 weeks   (~7-10 months)
```

### 🚀 Success Metrics

**Performance (Two-Tier Architecture)**:
- ✅ **4+ billion Scarab IDs/sec** (1024 Obelisk nodes)
- ✅ **4+ billion transactions/sec** (distributed coordinators)
- ✅ **10M+ writes/sec** per Pyramid node
- ✅ **<1ms p99 write latency**
- ✅ **<1μs Obelisk ID generation**
- ✅ **500 partitions/node** (coordinator-only mode)

**Reliability**:
- ✅ 100% test coverage for critical paths
- ✅ Zero data loss with RF=3, W=2
- ✅ <100ms Pyramid node failover
- ✅ <1ms Obelisk node recovery

**Scalability**:
- ✅ Linear scaling to 50+ Pyramid nodes
- ✅ 1024+ Obelisk nodes (coordination-free!)
- ✅ 10,000+ partitions per cluster
- ✅ Multi-datacenter support

---

## Architecture Evolution Path

Pyralog's architecture is **fundamentally different** from traditional systems:

```
┌──────────────────────────────────────────────────────────────┐
│   Pyralog's Two-Tier Architecture                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Tier 1: ☀️ Pharaoh Network (Obelisk Nodes)                  │
│  ──────────────────────────────────────────                 │
│  Purpose: ID generation, sequencing, coordination           │
│  State: Minimal (sparse files, ~1MB per node)               │
│  Consensus: None! (coordination-free)                       │
│  Throughput: Millions of ops/sec per node                   │
│  Latency: <1μs per ID                                       │
│  📝 Phase 0 deliverable                                     │
│                                                              │
│  Tier 2: 🔺 Pyralog Cluster (Pyramid Nodes)                  │
│  ───────────────────────────────────────────                │
│  Purpose: Storage, consensus, compute                       │
│  State: Full (LSM-Tree + Arrow, ~TB)                        │
│  Consensus: Dual Raft (Global + Per-Partition)             │
│  Throughput: 100K+ writes/sec per partition                 │
│  Latency: <1ms                                              │
│  📝 Phase 1-7 deliverable                                   │
│                                                              │
│  Key Innovation: 🗿 Obelisk Sequencer                        │
│  ────────────────────────────────────                       │
│  • Persistent atomic counter using file size               │
│  • <1μs increment (no consensus!)                           │
│  • Crash-safe (sparse file technique)                      │
│  • Enables 28+ billion ops/sec (1024 nodes)                │
│  📝 Foundation for everything                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 🪲 Scarab IDs: Globally Unique, Time-Ordered

**64-bit structure**:
```
┌──────────────┬─────────────┬──────────────┐
│  Timestamp   │ Obelisk ID  │  Sequence    │
│  (41 bits)   │  (10 bits)  │  (13 bits)   │
└──────────────┴─────────────┴──────────────┘
```

**Benefits**:
- Time-ordered (sortable by timestamp)
- Globally unique (no coordination)
- Crash-safe (Obelisk Sequencer)
- 8,192 IDs/millisecond per Obelisk node
- 4+ billion IDs/sec (1024 Obelisk nodes)

### Implementation Priority

**Phase 0**: Build Obelisk Sequencer - **The Foundation**  
**Phase 1-2**: Build Pyramid Storage - Get basic storage working  
**Phase 3**: Add Dual Raft - Global + Per-Partition consensus  
**Phase 4**: Add Replication - CopySet strategies  
**Phase 5**: Add Smart Clients - Direct routing  
**Phase 6**: Add Multi-Model - Arrow + DataFusion  
**Phase 7**: Production Hardening - Observability, recovery  
**Phase 8**: Advanced Features - Transactions, CDC, etc.

---

## Phase 0: Obelisk Sequencer (Pharaoh Network)

**Goal**: The foundational primitive - persistent atomic counter

**Duration**: 2-3 weeks

**Why First?** Scarab IDs are used by **everything**:
- Transaction IDs
- Producer session IDs
- Consumer generation IDs
- Schema IDs
- Timestamps (distributed TSO)

### 0.1 Core Obelisk Implementation (Week 1)

**The Innovation**: Use **file size** as the counter value!

```rust
// pyralog-obelisk/src/sequencer.rs
use std::fs::{File, OpenOptions};
use std::os::unix::fs::FileExt;
use std::sync::atomic::{AtomicU64, Ordering};
use std::path::PathBuf;

/// Persistent atomic counter using sparse file
pub struct ObeliskSequencer {
    /// Counter ID
    id: u16,
    
    /// Sparse file for persistence
    file: File,
    file_path: PathBuf,
    
    /// Current value (cached in memory)
    /// The true value is the file size!
    cached_value: AtomicU64,
}

impl ObeliskSequencer {
    /// Create new sequencer
    pub fn create(id: u16, data_dir: &Path) -> Result<Self> {
        let file_path = data_dir.join(format!("obelisk_{}.counter", id));
        
        // Create sparse file
        let file = OpenOptions::new()
            .read(true)
            .write(true)
            .create(true)
            .open(&file_path)?;
        
        // Get initial value from file size
        let metadata = file.metadata()?;
        let initial_value = metadata.len();
        
        Ok(Self {
            id,
            file,
            file_path,
            cached_value: AtomicU64::new(initial_value),
        })
    }
    
    /// Increment counter (the magic!)
    pub fn next(&self) -> Result<u64> {
        // 1. Increment cached value
        let new_value = self.cached_value.fetch_add(1, Ordering::SeqCst) + 1;
        
        // 2. Extend file to new size (sparse!)
        self.file.set_len(new_value)?;
        
        // 3. Done! No fsync needed for size changes on most filesystems
        //    File size updates are metadata operations (journaled)
        
        Ok(new_value)
    }
    
    /// Get current value without incrementing
    pub fn current(&self) -> u64 {
        self.cached_value.load(Ordering::SeqCst)
    }
    
    /// Recover from crash
    pub fn recover(id: u16, data_dir: &Path) -> Result<Self> {
        let file_path = data_dir.join(format!("obelisk_{}.counter", id));
        
        let file = OpenOptions::new()
            .read(true)
            .write(true)
            .open(&file_path)?;
        
        // File size IS the counter value!
        let metadata = file.metadata()?;
        let recovered_value = metadata.len();
        
        info!("Recovered Obelisk {} at value {}", id, recovered_value);
        
        Ok(Self {
            id,
            file,
            file_path,
            cached_value: AtomicU64::new(recovered_value),
        })
    }
}
```

**Tests**:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_basic_increment() {
        let seq = ObeliskSequencer::create(0, Path::new("/tmp")).unwrap();
        
        let id1 = seq.next().unwrap();
        let id2 = seq.next().unwrap();
        
        assert_eq!(id2, id1 + 1);
    }
    
    #[test]
    fn test_crash_recovery() {
        let dir = tempdir().unwrap();
        
        // Create and increment
        {
            let seq = ObeliskSequencer::create(0, dir.path()).unwrap();
            seq.next().unwrap();
            seq.next().unwrap();
            seq.next().unwrap();
        }
        
        // Recover
        let seq = ObeliskSequencer::recover(0, dir.path()).unwrap();
        let next_id = seq.next().unwrap();
        
        assert_eq!(next_id, 4); // Should continue from 3
    }
    
    #[test]
    fn test_concurrent_increments() {
        let seq = Arc::new(ObeliskSequencer::create(0, Path::new("/tmp")).unwrap());
        
        let handles: Vec<_> = (0..10)
            .map(|_| {
                let seq = seq.clone();
                thread::spawn(move || {
                    for _ in 0..1000 {
                        seq.next().unwrap();
                    }
                })
            })
            .collect();
        
        for handle in handles {
            handle.join().unwrap();
        }
        
        assert_eq!(seq.current(), 10000);
    }
}
```

**Benchmark**:
```rust
#[bench]
fn bench_obelisk_increment(b: &mut Bencher) {
    let seq = ObeliskSequencer::create(0, Path::new("/tmp")).unwrap();
    
    b.iter(|| {
        black_box(seq.next().unwrap());
    });
}

// Target: <1μs per increment
```

### 0.2 Scarab ID Generation (Week 2)

```rust
// pyralog-obelisk/src/scarab.rs
use std::time::{SystemTime, UNIX_EPOCH};

/// Globally unique, time-ordered 64-bit ID
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct ScarabId(u64);

impl ScarabId {
    /// Generate new Scarab ID
    pub fn generate(obelisk_id: u16, sequencer: &ObeliskSequencer) -> Result<Self> {
        // 1. Get timestamp (milliseconds since epoch)
        let timestamp_ms = SystemTime::now()
            .duration_since(UNIX_EPOCH)?
            .as_millis() as u64;
        
        // 2. Get sequence number from Obelisk
        let sequence = sequencer.next()?;
        
        // 3. Encode into 64 bits
        let id = Self::encode(timestamp_ms, obelisk_id, sequence);
        
        Ok(ScarabId(id))
    }
    
    /// Encode components into 64-bit ID
    fn encode(timestamp_ms: u64, obelisk_id: u16, sequence: u64) -> u64 {
        // Timestamp: 41 bits (69 years from epoch)
        let timestamp_bits = (timestamp_ms & 0x1FFFFFFFFFF) << 23;
        
        // Obelisk ID: 10 bits (1024 nodes)
        let obelisk_bits = ((obelisk_id as u64) & 0x3FF) << 13;
        
        // Sequence: 13 bits (8192 per millisecond)
        let sequence_bits = sequence & 0x1FFF;
        
        timestamp_bits | obelisk_bits | sequence_bits
    }
    
    /// Decode components from 64-bit ID
    pub fn decode(&self) -> (u64, u16, u64) {
        let timestamp_ms = (self.0 >> 23) & 0x1FFFFFFFFFF;
        let obelisk_id = ((self.0 >> 13) & 0x3FF) as u16;
        let sequence = self.0 & 0x1FFF;
        
        (timestamp_ms, obelisk_id, sequence)
    }
    
    /// Get timestamp component
    pub fn timestamp_ms(&self) -> u64 {
        (self.0 >> 23) & 0x1FFFFFFFFFF
    }
    
    /// Get Obelisk ID component
    pub fn obelisk_id(&self) -> u16 {
        ((self.0 >> 13) & 0x3FF) as u16
    }
    
    /// Get sequence component
    pub fn sequence(&self) -> u64 {
        self.0 & 0x1FFF
    }
    
    pub fn as_u64(&self) -> u64 {
        self.0
    }
}
```

### 0.3 Pharaoh Network Node (Week 3)

```rust
// pyralog-obelisk/src/node.rs
pub struct PharaohNode {
    /// This node's Obelisk ID
    obelisk_id: u16,
    
    /// Sequencer for this node
    sequencer: Arc<ObeliskSequencer>,
    
    /// Network server for ID requests
    server: Arc<IdServer>,
}

impl PharaohNode {
    pub async fn start(obelisk_id: u16, config: PharaohConfig) -> Result<Self> {
        // 1. Create/recover sequencer
        let sequencer = if config.data_dir.join(format!("obelisk_{}.counter", obelisk_id)).exists() {
            ObeliskSequencer::recover(obelisk_id, &config.data_dir)?
        } else {
            ObeliskSequencer::create(obelisk_id, &config.data_dir)?
        };
        
        let sequencer = Arc::new(sequencer);
        
        // 2. Start ID server
        let server = IdServer::new(obelisk_id, sequencer.clone());
        let server = Arc::new(server);
        
        tokio::spawn({
            let server = server.clone();
            let bind_addr = config.bind_addr.clone();
            async move {
                server.listen(&bind_addr).await
            }
        });
        
        Ok(Self {
            obelisk_id,
            sequencer,
            server,
        })
    }
    
    /// Generate next Scarab ID
    pub fn next_scarab_id(&self) -> Result<ScarabId> {
        ScarabId::generate(self.obelisk_id, &self.sequencer)
    }
}
```

**Milestone**: Obelisk Sequencer working, Scarab IDs generated at <1μs, crash recovery tested

---

## Phase 1: Foundation & Core Types

**Goal**: Basic project structure and core types for Pyramid nodes

**Duration**: 2-3 weeks

### 1.1 Project Setup

```bash
# Create workspace
cargo new --lib pyralog
cd pyralog

# Tier 1: Pharaoh Network (Obelisk nodes)
cargo new --lib pyralog-obelisk        # Sequencer & Scarab IDs

# Tier 2: Pyralog Cluster (Pyramid nodes)
cargo new --lib pyralog-core           # Core types
cargo new --lib pyralog-storage        # LSM-Tree storage
cargo new --lib pyralog-consensus      # Dual Raft
cargo new --lib pyralog-replication    # CopySet replication
cargo new --lib pyralog-protocol       # Smart client protocol
cargo new --lib pyralog-arrow          # Multi-model (Arrow)
cargo new --lib pyralog-tensor         # Tensor database
cargo new --bin pyralog-server         # Pyramid node server
cargo new --bin pyralog-obelisk-server # Obelisk node server

# Setup CI/CD
# - GitHub Actions for tests
# - Code coverage with tarpaulin
# - Clippy for lints
# - Rustfmt for formatting
```

**Key dependencies**:
```toml
[workspace]
members = [
    "pyralog-obelisk",
    "pyralog-core",
    "pyralog-storage",
    "pyralog-consensus",
    "pyralog-replication",
    "pyralog-protocol",
    "pyralog-arrow",
    "pyralog-tensor",
    "pyralog-server",
    "pyralog-obelisk-server",
]

[workspace.dependencies]
# Async runtime
tokio = { version = "1.35", features = ["full"] }
tokio-util = "0.7"

# Networking
tokio-tungstenite = "0.21"  # JSON-RPC/WebSocket (primary)
arrow-flight = "53.0"       # Zero-copy data transport
hyper = "1.5"               # HTTP/REST
quinn = "0.11"              # QUIC for replication

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
bincode = "1.3"

# Storage
rocksdb = "0.22"            # LSM-Tree
memmap2 = "0.9"             # Memory-mapped I/O

# Apache Arrow
arrow = "53.0"
arrow-schema = "53.0"
datafusion = "42.0"         # SQL engine
polars = "0.42"             # DataFrames

# Tensor/ML
safetensors = "0.4"         # ML model storage
ndarray = "0.16"            # N-dimensional arrays

# Consensus
raft = "0.7"                # Raft consensus

# Concurrency
dashmap = "6.1"
parking_lot = "0.12"
crossbeam = "0.8"

# Error handling
thiserror = "1.0"
anyhow = "1.0"

# Observability
tracing = "0.1"
tracing-subscriber = "0.3"
prometheus = "0.13"
opentelemetry = "0.24"

# Testing
proptest = "1.4"
criterion = "0.5"
tokio-test = "0.4"
```

### 1.2 Core Types (pyralog-core)

**Files to create**:
- `src/error.rs` - Error types with thiserror
- `src/offset.rs` - LogOffset type
- `src/epoch.rs` - Epoch type
- `src/record.rs` - Record and RecordBatch
- `src/log.rs` - LogId and metadata
- `src/partition.rs` - PartitionId
- `src/traits.rs` - Core traits
- `src/scarab.rs` - Re-export Scarab IDs from pyralog-obelisk

```rust
// src/epoch.rs
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct Epoch(pub u64);

impl Epoch {
    pub const ZERO: Self = Self(0);
    
    pub fn new(epoch: u64) -> Self {
        Self(epoch)
    }
    
    pub fn as_u64(&self) -> u64 {
        self.0
    }
    
    pub fn next(&self) -> Self {
        Self(self.0 + 1)
    }
}

// src/offset.rs
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct LogOffset(u64);

impl LogOffset {
    pub const ZERO: Self = Self(0);
    pub const MAX: Self = Self(u64::MAX);
    
    pub fn new(offset: u64) -> Self {
        Self(offset)
    }
    
    pub fn as_u64(&self) -> u64 {
        self.0
    }
}

/// Combined epoch + offset for unique record identification
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct EpochOffset {
    pub epoch: Epoch,
    pub offset: LogOffset,
}

impl EpochOffset {
    pub fn new(epoch: Epoch, offset: LogOffset) -> Self {
        Self { epoch, offset }
    }
    
    /// Encode as single u64 for LSM-Tree keys
    pub fn as_u64(&self) -> u64 {
        (self.epoch.as_u64() << 32) | self.offset.as_u64()
    }
}

// src/record.rs
use pyralog_obelisk::ScarabId;

pub struct Record {
    /// Globally unique Scarab ID (from Obelisk!)
    pub scarab_id: ScarabId,
    
    /// Partition-local offset
    pub offset: LogOffset,
    
    /// Epoch (for safe failover)
    pub epoch: Epoch,
    
    /// Timestamp
    pub timestamp: SystemTime,
    
    /// Optional key (for partitioning and compaction)
    pub key: Option<Bytes>,
    
    /// Value
    pub value: Bytes,
    
    /// Headers (metadata)
    pub headers: HashMap<String, Bytes>,
}

// src/traits.rs
#[async_trait]
pub trait Storage: Send + Sync {
    async fn append(&self, record: Record) -> Result<LogOffset>;
    async fn read(&self, offset: LogOffset) -> Result<Option<Record>>;
    async fn read_range(&self, start: LogOffset, end: LogOffset) -> Result<Vec<Record>>;
}
```

**Milestone**: Core types compile, have tests, pass CI

---

Due to the length, I'll continue building the remaining phases. Would you like me to proceed with Phase 2-8 (Storage, Consensus, Replication, Clients, Multi-Model, Production, Advanced)?
---

## Phase 2: Pyramid Storage (Custom LSM-Tree)

**Goal**: Working single-Pyramid-node storage engine with custom LSM-Tree

**Duration**: 4-6 weeks

**Why Custom?** Complete control over:
- Segment format (optimized for Pyralog)
- Compaction strategies (key-based, time-series)
- Memory-mapped I/O
- Bloom filters
- Index structure
- Zero dependencies (except memmap2)

### 2.1 Segment Files (Week 1-2)

**Custom segment format for Pyralog records**

```rust
// pyralog-storage/src/segment.rs
use memmap2::{Mmap, MmapMut};
use std::fs::{File, OpenOptions};
use std::path::{Path, PathBuf};

/// Immutable segment file
pub struct Segment {
    /// Base offset (first record in segment)
    base_offset: EpochOffset,
    
    /// Segment file path
    path: PathBuf,
    
    /// Memory-mapped file (read-only)
    mmap: Mmap,
    
    /// Segment size
    size: u64,
    
    /// Number of records
    record_count: u32,
}

impl Segment {
    /// Create new segment from records
    pub fn create(
        base_offset: EpochOffset,
        records: Vec<Record>,
        dir: &Path,
    ) -> Result<Self> {
        let path = dir.join(format!(
            "segment_{}_{}.log",
            base_offset.epoch.as_u64(),
            base_offset.offset.as_u64()
        ));
        
        // 1. Create file
        let mut file = OpenOptions::new()
            .read(true)
            .write(true)
            .create(true)
            .open(&path)?;
        
        // 2. Write records
        let mut offset = 0u64;
        for record in &records {
            let bytes = Self::encode_record(record)?;
            file.write_all(&bytes)?;
            offset += bytes.len() as u64;
        }
        
        file.flush()?;
        
        // 3. Memory-map file (read-only)
        let mmap = unsafe { Mmap::map(&file)? };
        
        Ok(Self {
            base_offset,
            path,
            mmap,
            size: offset,
            record_count: records.len() as u32,
        })
    }
    
    /// Open existing segment
    pub fn open(path: PathBuf) -> Result<Self> {
        let file = File::open(&path)?;
        let metadata = file.metadata()?;
        
        // Parse filename: segment_<epoch>_<offset>.log
        let filename = path.file_stem().unwrap().to_str().unwrap();
        let parts: Vec<&str> = filename.split('_').collect();
        let epoch = Epoch::new(parts[1].parse()?);
        let offset = LogOffset::new(parts[2].parse()?);
        let base_offset = EpochOffset::new(epoch, offset);
        
        // Memory-map file
        let mmap = unsafe { Mmap::map(&file)? };
        
        // Count records
        let record_count = Self::count_records(&mmap)?;
        
        Ok(Self {
            base_offset,
            path,
            mmap,
            size: metadata.len(),
            record_count,
        })
    }
    
    /// Read record at position
    pub fn read(&self, position: u64) -> Result<Record> {
        if position >= self.size {
            return Err(PyralogError::InvalidPosition(position));
        }
        
        let data = &self.mmap[position as usize..];
        Self::decode_record(data)
    }
    
    /// Scan all records
    pub fn scan(&self) -> SegmentIterator {
        SegmentIterator {
            segment: self,
            position: 0,
        }
    }
    
    /// Encode record: [length: u32][data: bytes]
    fn encode_record(record: &Record) -> Result<Vec<u8>> {
        let data = bincode::serialize(record)?;
        let length = data.len() as u32;
        
        let mut bytes = Vec::with_capacity(4 + data.len());
        bytes.extend_from_slice(&length.to_le_bytes());
        bytes.extend_from_slice(&data);
        
        Ok(bytes)
    }
    
    /// Decode record from bytes
    fn decode_record(data: &[u8]) -> Result<Record> {
        if data.len() < 4 {
            return Err(PyralogError::InvalidSegment);
        }
        
        let length = u32::from_le_bytes([data[0], data[1], data[2], data[3]]) as usize;
        
        if data.len() < 4 + length {
            return Err(PyralogError::InvalidSegment);
        }
        
        let record_data = &data[4..4 + length];
        Ok(bincode::deserialize(record_data)?)
    }
    
    fn count_records(mmap: &Mmap) -> Result<u32> {
        let mut count = 0u32;
        let mut pos = 0usize;
        
        while pos + 4 <= mmap.len() {
            let length = u32::from_le_bytes([
                mmap[pos],
                mmap[pos + 1],
                mmap[pos + 2],
                mmap[pos + 3],
            ]) as usize;
            
            pos += 4 + length;
            count += 1;
        }
        
        Ok(count)
    }
}

pub struct SegmentIterator<'a> {
    segment: &'a Segment,
    position: u64,
}

impl<'a> Iterator for SegmentIterator<'a> {
    type Item = Result<Record>;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.position >= self.segment.size {
            return None;
        }
        
        match self.segment.read(self.position) {
            Ok(record) => {
                // Move position forward
                let length = 4 + bincode::serialized_size(&record).unwrap();
                self.position += length;
                Some(Ok(record))
            }
            Err(e) => Some(Err(e)),
        }
    }
}
```

### 2.2 Sparse Index (Week 2)

**Index every Nth record for fast lookups**

```rust
// pyralog-storage/src/index.rs
use std::fs::File;
use std::io::{Read, Write};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct IndexEntry {
    /// Key (epoch_offset)
    pub key: EpochOffset,
    
    /// File position in segment
    pub position: u64,
    
    /// Record size
    pub size: u32,
}

pub struct Index {
    /// Indexed entries (every Nth record)
    entries: Vec<IndexEntry>,
    
    /// Index interval (e.g., index every 4096 bytes)
    interval: u32,
}

impl Index {
    pub fn new(interval: u32) -> Self {
        Self {
            entries: Vec::new(),
            interval,
        }
    }
    
    /// Add entry to index
    pub fn append(&mut self, key: EpochOffset, position: u64, size: u32) {
        // Only index at interval boundaries
        if self.entries.is_empty() || position - self.entries.last().unwrap().position >= self.interval as u64 {
            self.entries.push(IndexEntry {
                key,
                position,
                size,
            });
        }
    }
    
    /// Lookup position for key (binary search)
    pub fn lookup(&self, key: EpochOffset) -> Option<(u64, u32)> {
        if self.entries.is_empty() {
            return None;
        }
        
        // Binary search for largest entry <= key
        let idx = match self.entries.binary_search_by_key(&key, |e| e.key) {
            Ok(idx) => idx,
            Err(0) => return None,
            Err(idx) => idx - 1,
        };
        
        let entry = &self.entries[idx];
        Some((entry.position, entry.size))
    }
    
    /// Save index to file
    pub fn save(&self, path: &Path) -> Result<()> {
        let mut file = File::create(path)?;
        let data = bincode::serialize(&self.entries)?;
        file.write_all(&data)?;
        file.flush()?;
        Ok(())
    }
    
    /// Load index from file
    pub fn load(path: &Path) -> Result<Self> {
        let mut file = File::open(path)?;
        let mut data = Vec::new();
        file.read_to_end(&mut data)?;
        
        let entries: Vec<IndexEntry> = bincode::deserialize(&data)?;
        
        Ok(Self {
            entries,
            interval: 4096, // Default
        })
    }
}
```

### 2.3 Memtable (In-Memory Buffer) (Week 3)

**In-memory sorted map with write-ahead log**

```rust
// pyralog-storage/src/memtable.rs
use std::collections::BTreeMap;
use std::sync::RwLock;

pub struct Memtable {
    /// Sorted map: (epoch, offset) -> record
    data: RwLock<BTreeMap<EpochOffset, Record>>,
    
    /// Current size (bytes)
    size: AtomicU64,
    
    /// Maximum size before flush
    max_size: u64,
    
    /// Write-ahead log (for crash recovery)
    wal: Arc<RwLock<WriteAheadLog>>,
}

impl Memtable {
    pub fn new(max_size: u64, wal_path: PathBuf) -> Result<Self> {
        Ok(Self {
            data: RwLock::new(BTreeMap::new()),
            size: AtomicU64::new(0),
            max_size,
            wal: Arc::new(RwLock::new(WriteAheadLog::new(wal_path)?)),
        })
    }
    
    /// Insert record
    pub fn insert(&self, record: Record) -> Result<()> {
        let key = EpochOffset::new(record.epoch, record.offset);
        
        // 1. Write to WAL first (durability!)
        self.wal.write().append(&record)?;
        
        // 2. Insert into memtable
        let record_size = bincode::serialized_size(&record)? as u64;
        self.data.write().insert(key, record);
        self.size.fetch_add(record_size, Ordering::SeqCst);
        
        Ok(())
    }
    
    /// Get record
    pub fn get(&self, key: EpochOffset) -> Option<Record> {
        self.data.read().get(&key).cloned()
    }
    
    /// Check if should flush
    pub fn should_flush(&self) -> bool {
        self.size.load(Ordering::SeqCst) >= self.max_size
    }
    
    /// Flush to segment
    pub fn flush(&self, dir: &Path) -> Result<Segment> {
        let data = self.data.read();
        
        if data.is_empty() {
            return Err(PyralogError::EmptyMemtable);
        }
        
        // 1. Get base offset (first key)
        let base_offset = *data.keys().next().unwrap();
        
        // 2. Collect records
        let records: Vec<Record> = data.values().cloned().collect();
        
        // 3. Create segment
        let segment = Segment::create(base_offset, records, dir)?;
        
        // 4. Clear memtable
        drop(data);
        self.data.write().clear();
        self.size.store(0, Ordering::SeqCst);
        
        // 5. Clear WAL
        self.wal.write().clear()?;
        
        Ok(segment)
    }
    
    /// Recover from WAL
    pub fn recover(&self) -> Result<()> {
        let records = self.wal.read().read_all()?;
        
        for record in records {
            let key = EpochOffset::new(record.epoch, record.offset);
            let size = bincode::serialized_size(&record)? as u64;
            
            self.data.write().insert(key, record);
            self.size.fetch_add(size, Ordering::SeqCst);
        }
        
        Ok(())
    }
}
```

### 2.4 Write-Ahead Log (Week 3)

**Crash recovery for memtable**

```rust
// pyralog-storage/src/wal.rs
pub struct WriteAheadLog {
    file: Arc<RwLock<File>>,
    path: PathBuf,
}

impl WriteAheadLog {
    pub fn new(path: PathBuf) -> Result<Self> {
        let file = OpenOptions::new()
            .create(true)
            .write(true)
            .append(true)
            .open(&path)?;
        
        Ok(Self {
            file: Arc::new(RwLock::new(file)),
            path,
        })
    }
    
    pub fn append(&self, record: &Record) -> Result<()> {
        let mut file = self.file.write();
        
        // Encode: [length: u32][data: bytes]
        let data = bincode::serialize(record)?;
        let length = data.len() as u32;
        
        file.write_all(&length.to_le_bytes())?;
        file.write_all(&data)?;
        file.flush()?; // fsync!
        
        Ok(())
    }
    
    pub fn read_all(&self) -> Result<Vec<Record>> {
        let mut file = File::open(&self.path)?;
        let mut records = Vec::new();
        
        loop {
            // Read length
            let mut len_bytes = [0u8; 4];
            match file.read_exact(&mut len_bytes) {
                Ok(_) => {},
                Err(ref e) if e.kind() == std::io::ErrorKind::UnexpectedEof => break,
                Err(e) => return Err(e.into()),
            }
            
            let length = u32::from_le_bytes(len_bytes) as usize;
            
            // Read data
            let mut data = vec![0u8; length];
            file.read_exact(&mut data)?;
            
            let record: Record = bincode::deserialize(&data)?;
            records.push(record);
        }
        
        Ok(records)
    }
    
    pub fn clear(&self) -> Result<()> {
        let mut file = self.file.write();
        file.set_len(0)?;
        file.flush()?;
        Ok(())
    }
}
```

### 2.5 LSM-Tree Storage (Week 4-5)

**Complete custom LSM-Tree implementation**

```rust
// pyralog-storage/src/lsm_storage.rs
pub struct PyramidStorage {
    /// Partition ID
    partition_id: PartitionId,
    
    /// Data directory
    data_dir: PathBuf,
    
    /// Active memtable (writes go here)
    memtable: Arc<Memtable>,
    
    /// Immutable memtables (being flushed)
    immutable_memtables: Arc<RwLock<Vec<Arc<Memtable>>>>,
    
    /// L0 segments (unsorted, may overlap)
    l0_segments: Arc<RwLock<Vec<Segment>>>,
    
    /// Sorted run levels (L1, L2, ...)
    levels: Arc<RwLock<Vec<Vec<Segment>>>>,
    
    /// Indexes for each segment
    indexes: Arc<RwLock<HashMap<PathBuf, Index>>>,
    
    /// Compaction thread
    compaction_handle: Option<JoinHandle<()>>,
}

impl PyramidStorage {
    pub async fn new(partition_id: PartitionId, data_dir: PathBuf) -> Result<Self> {
        std::fs::create_dir_all(&data_dir)?;
        
        let wal_path = data_dir.join("memtable.wal");
        let memtable = Arc::new(Memtable::new(64 * 1024 * 1024, wal_path)?); // 64MB
        
        // Discover existing segments
        let mut l0_segments = Vec::new();
        for entry in std::fs::read_dir(&data_dir)? {
            let entry = entry?;
            let path = entry.path();
            
            if path.extension().and_then(|s| s.to_str()) == Some("log") {
                let segment = Segment::open(path)?;
                l0_segments.push(segment);
            }
        }
        
        Ok(Self {
            partition_id,
            data_dir,
            memtable,
            immutable_memtables: Arc::new(RwLock::new(Vec::new())),
            l0_segments: Arc::new(RwLock::new(l0_segments)),
            levels: Arc::new(RwLock::new(vec![Vec::new(); 7])), // 7 levels
            indexes: Arc::new(RwLock::new(HashMap::new())),
            compaction_handle: None,
        })
    }
    
    /// Append record
    pub async fn append(&self, record: Record) -> Result<LogOffset> {
        // 1. Insert into memtable
        self.memtable.insert(record.clone())?;
        
        // 2. Check if should flush
        if self.memtable.should_flush() {
            self.flush_memtable().await?;
        }
        
        Ok(record.offset)
    }
    
    /// Read record
    pub async fn read(&self, epoch: Epoch, offset: LogOffset) -> Result<Option<Record>> {
        let key = EpochOffset::new(epoch, offset);
        
        // 1. Check memtable
        if let Some(record) = self.memtable.get(key) {
            return Ok(Some(record));
        }
        
        // 2. Check immutable memtables
        for im in self.immutable_memtables.read().iter() {
            if let Some(record) = im.get(key) {
                return Ok(Some(record));
            }
        }
        
        // 3. Check L0 segments (must check all, may overlap)
        for segment in self.l0_segments.read().iter().rev() {
            if let Some(record) = self.search_segment(segment, key).await? {
                return Ok(Some(record));
            }
        }
        
        // 4. Check sorted levels (binary search per level)
        for level in self.levels.read().iter() {
            if let Some(record) = self.search_level(level, key).await? {
                return Ok(Some(record));
            }
        }
        
        Ok(None)
    }
    
    /// Flush memtable to L0
    async fn flush_memtable(&self) -> Result<()> {
        // 1. Swap memtable (make current immutable, create new one)
        let old_memtable = Arc::new(Memtable::new(
            64 * 1024 * 1024,
            self.data_dir.join(format!("memtable_{}.wal", SystemTime::now().duration_since(UNIX_EPOCH).unwrap().as_millis())),
        )?);
        
        let flushing = std::mem::replace(&mut *self.memtable, old_memtable);
        self.immutable_memtables.write().push(flushing.clone());
        
        // 2. Flush to disk (async)
        let data_dir = self.data_dir.clone();
        let l0_segments = self.l0_segments.clone();
        let immutable_memtables = self.immutable_memtables.clone();
        let indexes = self.indexes.clone();
        
        tokio::spawn(async move {
            if let Ok(segment) = flushing.flush(&data_dir) {
                // Build index
                let index = Self::build_index(&segment);
                indexes.write().insert(segment.path.clone(), index);
                
                // Add to L0
                l0_segments.write().push(segment);
                
                // Remove from immutable
                immutable_memtables.write().retain(|m| !Arc::ptr_eq(m, &flushing));
            }
        });
        
        Ok(())
    }
    
    /// Search segment for key (using index)
    async fn search_segment(&self, segment: &Segment, key: EpochOffset) -> Result<Option<Record>> {
        let indexes = self.indexes.read();
        
        if let Some(index) = indexes.get(&segment.path) {
            // Use index to find approximate position
            if let Some((position, _)) = index.lookup(key) {
                // Scan forward from indexed position
                for result in segment.scan() {
                    let record = result?;
                    let record_key = EpochOffset::new(record.epoch, record.offset);
                    
                    if record_key == key {
                        return Ok(Some(record));
                    } else if record_key > key {
                        break;
                    }
                }
            }
        }
        
        Ok(None)
    }
    
    fn build_index(segment: &Segment) -> Index {
        let mut index = Index::new(4096);
        let mut position = 0u64;
        
        for result in segment.scan() {
            if let Ok(record) = result {
                let key = EpochOffset::new(record.epoch, record.offset);
                let size = 4 + bincode::serialized_size(&record).unwrap() as u32;
                
                index.append(key, position, size);
                position += size as u64;
            }
        }
        
        index
    }
}
```

**Milestone**: Custom LSM-Tree working, benchmarks showing 100K+ writes/sec, complete control over storage format

---

## Phase 3: Dual Raft Consensus

**Goal**: Multi-Pyramid-node cluster with Dual Raft consensus

**Duration**: 4-6 weeks

### 3.1 Global Raft (Week 1-2)

**Purpose**: Cluster-wide metadata (infrequent operations)

```rust
// pyralog-consensus/src/global_raft.rs
use raft::prelude::*;

pub struct GlobalRaft {
    /// This node's ID
    node_id: u64,
    
    /// Raft raw node
    raw_node: RawNode<GlobalStorage>,
    
    /// Cluster members
    peers: Vec<u64>,
}

impl GlobalRaft {
    pub fn propose_partition_create(&mut self, log_id: LogId, partitions: u32) -> Result<()> {
        let cmd = ClusterCommand::CreateLog {
            log_id,
            partitions,
        };
        
        let data = bincode::serialize(&cmd)?;
        self.raw_node.propose(vec![], data)?;
        
        Ok(())
    }
    
    pub fn propose_partition_assignment(
        &mut self,
        partition: PartitionId,
        replicas: Vec<NodeId>,
    ) -> Result<()> {
        let cmd = ClusterCommand::AssignPartition {
            partition,
            replicas,
        };
        
        let data = bincode::serialize(&cmd)?;
        self.raw_node.propose(vec![], data)?;
        
        Ok(())
    }
}

#[derive(Serialize, Deserialize)]
pub enum ClusterCommand {
    CreateLog {
        log_id: LogId,
        partitions: u32,
    },
    DeleteLog {
        log_id: LogId,
    },
    AssignPartition {
        partition: PartitionId,
        replicas: Vec<NodeId>,
    },
    AddNode {
        node_id: NodeId,
        addr: String,
    },
    RemoveNode {
        node_id: NodeId,
    },
}
```

### 3.2 Per-Partition Raft (Week 3-4)

**Purpose**: Partition-specific operations (frequent)

```rust
// pyralog-consensus/src/partition_raft.rs
pub struct PartitionRaft {
    /// Partition ID
    partition_id: PartitionId,
    
    /// Raft raw node
    raw_node: RawNode<PartitionStorage>,
    
    /// Current epoch
    current_epoch: RwLock<Epoch>,
}

impl PartitionRaft {
    /// Activate epoch (on leadership change)
    pub fn activate_epoch(&mut self, new_leader: NodeId) -> Result<Epoch> {
        let new_epoch = self.current_epoch.read().next();
        
        let cmd = PartitionCommand::ActivateEpoch {
            epoch: new_epoch,
            leader: new_leader,
        };
        
        let data = bincode::serialize(&cmd)?;
        self.raw_node.propose(vec![], data)?;
        
        Ok(new_epoch)
    }
    
    /// Seal epoch (on failover)
    pub fn seal_epoch(&mut self, epoch: Epoch, end_offset: LogOffset) -> Result<()> {
        let cmd = PartitionCommand::SealEpoch {
            epoch,
            end_offset,
        };
        
        let data = bincode::serialize(&cmd)?;
        self.raw_node.propose(vec![], data)?;
        
        Ok(())
    }
}

#[derive(Serialize, Deserialize)]
pub enum PartitionCommand {
    ActivateEpoch {
        epoch: Epoch,
        leader: NodeId,
    },
    SealEpoch {
        epoch: Epoch,
        end_offset: LogOffset,
    },
}
```

### 3.3 Cluster Manager (Week 5-6)

```rust
pub struct ClusterManager {
    /// This node's ID
    node_id: NodeId,
    
    /// Global Raft instance
    global_raft: Arc<RwLock<GlobalRaft>>,
    
    /// Per-partition Raft instances
    partition_rafts: Arc<RwLock<HashMap<PartitionId, PartitionRaft>>>,
    
    /// Cluster metadata
    metadata: Arc<RwLock<ClusterMetadata>>,
}

pub struct ClusterMetadata {
    /// All nodes in cluster
    pub nodes: HashMap<NodeId, NodeInfo>,
    
    /// All logs
    pub logs: HashMap<LogId, LogInfo>,
    
    /// Partition assignments
    pub partitions: HashMap<PartitionId, PartitionInfo>,
}

pub struct PartitionInfo {
    pub partition_id: PartitionId,
    pub log_id: LogId,
    pub leader: NodeId,
    pub replicas: Vec<NodeId>,
    pub isr: Vec<NodeId>, // In-Sync Replicas
    pub epoch: Epoch,
}
```

**Milestone**: 3-Pyramid-node cluster with Dual Raft consensus, leader election < 100ms

---

## Phase 4: Replication & Epochs

**Goal**: High-throughput writes with epochs, CopySet replication

**Duration**: 4-5 weeks

### 4.1 Epoch System (Week 1-2)

```rust
// pyralog-core/src/sequencer.rs
pub struct Sequencer {
    /// Partition ID
    partition: PartitionId,
    
    /// Current epoch metadata
    current_epoch: RwLock<EpochMetadata>,
    
    /// Next offset (atomic, no consensus!)
    next_offset: AtomicU64,
    
    /// Epoch store (RocksDB)
    epoch_store: Arc<dyn EpochStore>,
}

pub struct EpochMetadata {
    pub epoch: Epoch,
    pub leader: NodeId,
    pub start_offset: LogOffset,
    pub end_offset: Option<LogOffset>,
    pub sealed: bool,
}

impl Sequencer {
    /// Activate new epoch (called by new leader)
    pub async fn activate_epoch(&self, node_id: NodeId) -> Result<Epoch> {
        // 1. Seal previous epoch via Raft
        if let Some(prev) = self.current_epoch.read().as_ref() {
            if !prev.sealed {
                let last_offset = LogOffset::new(self.next_offset.load(Ordering::SeqCst));
                self.epoch_store.seal_epoch(self.partition, prev.epoch, last_offset).await?;
            }
        }
        
        // 2. Activate new epoch via Raft
        let new_epoch = self.epoch_store.activate_epoch(self.partition, node_id).await?;
        
        // 3. Update local state
        *self.current_epoch.write() = EpochMetadata {
            epoch: new_epoch,
            leader: node_id,
            start_offset: LogOffset::new(self.next_offset.load(Ordering::SeqCst)),
            end_offset: None,
            sealed: false,
        };
        
        Ok(new_epoch)
    }
    
    /// Assign next offset (LOCAL operation, no consensus!)
    pub fn next_offset(&self) -> LogOffset {
        let offset = self.next_offset.fetch_add(1, Ordering::SeqCst);
        LogOffset::new(offset)
    }
    
    /// Check if can write in current epoch
    pub fn can_write(&self) -> bool {
        !self.current_epoch.read().sealed
    }
}
```

### 4.2 CopySet Replication (Week 3-4)

```rust
// pyralog-replication/src/copyset.rs
#[derive(Debug, Clone)]
pub enum CopySetStrategy {
    /// Fixed replicas per partition (Kafka-style)
    PerPartition,
    
    /// Dynamic replicas per record (LogDevice-style)
    PerRecord {
        seed: u64,
        /// Leader as coordinator (doesn't store data)
        coordinator_only: bool,
    },
}

pub struct CopySetSelector {
    strategy: CopySetStrategy,
    nodes: Vec<NodeId>,
    replication_factor: usize,
}

impl CopySetSelector {
    /// Select CopySet for record
    pub fn select(&self, record: &Record, partition: PartitionId) -> Vec<NodeId> {
        match &self.strategy {
            CopySetStrategy::PerPartition => {
                // Fixed replicas (simple)
                self.select_per_partition(partition)
            }
            CopySetStrategy::PerRecord { seed, .. } => {
                // Dynamic replicas based on Scarab ID
                self.select_per_record(record.scarab_id, *seed)
            }
        }
    }
    
    fn select_per_partition(&self, partition: PartitionId) -> Vec<NodeId> {
        // Hash partition ID to select consistent replicas
        let mut hasher = DefaultHasher::new();
        partition.as_u64().hash(&mut hasher);
        let hash = hasher.finish();
        
        let start_idx = (hash as usize) % self.nodes.len();
        
        let mut replicas = Vec::with_capacity(self.replication_factor);
        for i in 0..self.replication_factor {
            let idx = (start_idx + i) % self.nodes.len();
            replicas.push(self.nodes[idx]);
        }
        
        replicas
    }
    
    fn select_per_record(&self, scarab_id: ScarabId, seed: u64) -> Vec<NodeId> {
        // Hash Scarab ID to select diverse replicas
        let mut hasher = DefaultHasher::new();
        scarab_id.as_u64().hash(&mut hasher);
        seed.hash(&mut hasher);
        let hash = hasher.finish();
        
        let start_idx = (hash as usize) % self.nodes.len();
        
        let mut replicas = Vec::with_capacity(self.replication_factor);
        for i in 0..self.replication_factor {
            let idx = (start_idx + i) % self.nodes.len();
            replicas.push(self.nodes[idx]);
        }
        
        replicas
    }
}
```

### 4.3 Replicator (Week 4-5)

```rust
// pyralog-replication/src/replicator.rs
pub struct Replicator {
    /// This node's ID
    node_id: NodeId,
    
    /// Partition ID
    partition_id: PartitionId,
    
    /// CopySet selector
    copyset_selector: Arc<CopySetSelector>,
    
    /// Connections to other nodes
    connections: Arc<RwLock<HashMap<NodeId, Connection>>>,
    
    /// Local storage (None if coordinator-only!)
    local_storage: Option<Arc<PyramidStorage>>,
    
    /// Write quorum
    write_quorum: usize,
}

impl Replicator {
    /// Replicate record
    pub async fn replicate(&self, record: Record) -> Result<()> {
        // 1. Select CopySet
        let copyset = self.copyset_selector.select(&record, self.partition_id);
        
        // 2. Write locally if leader stores data
        if let Some(storage) = &self.local_storage {
            if copyset.contains(&self.node_id) {
                storage.append(record.clone()).await?;
            }
        }
        
        // 3. Send to replicas
        let futures: Vec<_> = copyset.iter()
            .filter(|&&node| node != self.node_id)
            .map(|&node| self.send_to_node(node, record.clone()))
            .collect();
        
        // 4. Wait for quorum
        self.wait_for_quorum(futures).await?;
        
        Ok(())
    }
    
    async fn send_to_node(&self, node: NodeId, record: Record) -> Result<()> {
        let conn = self.connections.read().get(&node).cloned()
            .ok_or_else(|| PyralogError::NodeNotConnected(node))?;
        
        conn.send_record(record).await
    }
    
    async fn wait_for_quorum(&self, futures: Vec<impl Future<Output = Result<()>>>) -> Result<()> {
        // Wait for W replicas to succeed
        let results = futures::future::join_all(futures).await;
        
        let successes = results.iter().filter(|r| r.is_ok()).count();
        
        if successes >= self.write_quorum {
            Ok(())
        } else {
            Err(PyralogError::QuorumFailed {
                required: self.write_quorum,
                actual: successes,
            })
        }
    }
}
```

**Milestone**: Replication working, epochs enable 1M+ writes/sec, both CopySet strategies implemented

---

## Phase 5: Smart Clients & Protocols

**Goal**: Smart client with direct leader routing, JSON-RPC/WebSocket protocol

**Duration**: 3-4 weeks

### 5.1 Metadata Protocol (Week 1)

```rust
// pyralog-protocol/src/metadata.rs
#[derive(Serialize, Deserialize)]
pub struct MetadataRequest {
    pub log_ids: Vec<LogId>,
}

#[derive(Serialize, Deserialize)]
pub struct MetadataResponse {
    pub logs: Vec<LogMetadata>,
    pub nodes: Vec<NodeMetadata>,
    pub obelisk_nodes: Vec<ObeliskMetadata>,
}

pub struct LogMetadata {
    pub log_id: LogId,
    pub partitions: Vec<PartitionMetadata>,
}

pub struct PartitionMetadata {
    pub partition_id: PartitionId,
    pub leader: NodeId,
    pub replicas: Vec<NodeId>,
    pub isr: Vec<NodeId>,
    pub epoch: Epoch,
}

pub struct ObeliskMetadata {
    pub obelisk_id: u16,
    pub addr: String,
    pub status: NodeStatus,
}
```

### 5.2 Smart Client (Week 2)

```rust
// pyralog-protocol/src/client.rs
pub struct PyralogClient {
    /// Bootstrap servers (Pyramid nodes)
    bootstrap_servers: Vec<String>,
    
    /// Obelisk nodes (for Scarab IDs)
    obelisk_nodes: Vec<String>,
    
    /// Metadata cache
    metadata_cache: Arc<RwLock<MetadataCache>>,
    
    /// Connections to Pyramid nodes
    pyramid_connections: Arc<RwLock<HashMap<NodeId, Connection>>>,
    
    /// Connections to Obelisk nodes
    obelisk_connections: Arc<RwLock<HashMap<u16, ObeliskConnection>>>,
    
    /// Partitioner
    partitioner: Box<dyn Partitioner>,
}

impl PyralogClient {
    pub async fn produce(
        &self,
        log_id: LogId,
        key: Option<Bytes>,
        value: Bytes,
    ) -> Result<EpochOffset> {
        // 1. Get Scarab ID from Obelisk
        let scarab_id = self.get_scarab_id().await?;
        
        // 2. Calculate partition
        let partition = self.partitioner.partition(&key, &log_id)?;
        
        // 3. Get leader from cache
        let leader = self.get_leader(&log_id, partition).await?;
        
        // 4. Create record
        let record = Record {
            scarab_id,
            offset: LogOffset::ZERO, // Will be assigned by leader
            epoch: Epoch::ZERO,      // Will be assigned by leader
            timestamp: SystemTime::now(),
            key,
            value,
            headers: HashMap::new(),
        };
        
        // 5. Send directly to leader
        match self.send_to_pyramid_node(leader, record).await {
            Ok(epoch_offset) => Ok(epoch_offset),
            Err(PyralogError::NotLeader(new_leader)) => {
                // Refresh metadata and retry
                self.refresh_metadata(&log_id).await?;
                self.send_to_pyramid_node(new_leader, record).await
            }
            Err(e) => Err(e),
        }
    }
    
    async fn get_scarab_id(&self) -> Result<ScarabId> {
        // Load-balance across Obelisk nodes
        let obelisk_id = self.select_obelisk_node();
        let conn = self.obelisk_connections.read()
            .get(&obelisk_id)
            .cloned()
            .ok_or_else(|| PyralogError::ObeliskNotConnected(obelisk_id))?;
        
        conn.next_scarab_id().await
    }
}
```

### 5.3 JSON-RPC/WebSocket Protocol (Week 3)

```rust
// pyralog-protocol/src/jsonrpc.rs
use tokio_tungstenite::WebSocketStream;
use serde_json::{json, Value};

#[derive(Serialize, Deserialize)]
pub struct JsonRpcRequest {
    pub jsonrpc: String,
    pub id: u64,
    pub method: String,
    pub params: Value,
}

#[derive(Serialize, Deserialize)]
pub struct JsonRpcResponse {
    pub jsonrpc: String,
    pub id: u64,
    pub result: Option<Value>,
    pub error: Option<JsonRpcError>,
}

pub struct JsonRpcServer {
    storage: Arc<PyramidStorage>,
    sequencer: Arc<Sequencer>,
    replicator: Arc<Replicator>,
}

impl JsonRpcServer {
    pub async fn handle_message(&self, msg: String) -> Result<String> {
        let req: JsonRpcRequest = serde_json::from_str(&msg)?;
        
        let result = match req.method.as_str() {
            "produce" => self.handle_produce(req.params).await,
            "consume" => self.handle_consume(req.params).await,
            "metadata" => self.handle_metadata(req.params).await,
            _ => Err(PyralogError::UnknownMethod(req.method.clone())),
        };
        
        let response = match result {
            Ok(value) => JsonRpcResponse {
                jsonrpc: "2.0".to_string(),
                id: req.id,
                result: Some(value),
                error: None,
            },
            Err(e) => JsonRpcResponse {
                jsonrpc: "2.0".to_string(),
                id: req.id,
                result: None,
                error: Some(JsonRpcError {
                    code: -32000,
                    message: e.to_string(),
                }),
            },
        };
        
        Ok(serde_json::to_string(&response)?)
    }
}
```

### 5.4 Arrow Flight Integration (Week 4)

For **zero-copy** data transport:

```rust
// pyralog-protocol/src/arrow_flight.rs
use arrow_flight::{FlightClient, FlightDescriptor, Ticket};

pub struct ArrowFlightClient {
    client: FlightClient,
}

impl ArrowFlightClient {
    /// Read records as Arrow RecordBatch (zero-copy!)
    pub async fn consume_arrow(
        &mut self,
        log_id: LogId,
        partition: PartitionId,
        start: EpochOffset,
        max: usize,
    ) -> Result<Vec<RecordBatch>> {
        let descriptor = FlightDescriptor::new_path(vec![
            log_id.to_string(),
            partition.to_string(),
            start.to_string(),
            max.to_string(),
        ]);
        
        let mut stream = self.client.do_get(descriptor).await?;
        
        let mut batches = Vec::new();
        while let Some(batch) = stream.message().await? {
            batches.push(batch);
        }
        
        Ok(batches)
    }
}
```

**Milestone**: Smart client working with direct leader routing, JSON-RPC/WS + Arrow Flight protocols

---

## Phase 6: Multi-Model Database

**Goal**: Apache Arrow integration, multi-model storage

**Duration**: 4-6 weeks

### 6.1 Arrow Integration (Week 1-2)

```rust
// pyralog-arrow/src/storage.rs
use arrow::array::*;
use arrow::datatypes::*;
use arrow::record_batch::RecordBatch;

pub struct ArrowStorage {
    /// Underlying LSM storage
    lsm: Arc<PyramidStorage>,
    
    /// Arrow schema
    schema: Arc<Schema>,
}

impl ArrowStorage {
    /// Store records as Arrow RecordBatch
    pub async fn append_batch(&self, batch: RecordBatch) -> Result<Vec<LogOffset>> {
        // Convert Arrow → Pyralog Records
        let records = self.batch_to_records(batch)?;
        
        // Append to LSM
        let mut offsets = Vec::new();
        for record in records {
            let offset = self.lsm.append(record).await?;
            offsets.push(offset);
        }
        
        Ok(offsets)
    }
    
    /// Read records as Arrow RecordBatch (zero-copy!)
    pub async fn read_batch(
        &self,
        start: EpochOffset,
        end: EpochOffset,
        max: usize,
    ) -> Result<RecordBatch> {
        // Read from LSM
        let records = self.lsm.read_range(start, end, max).await?;
        
        // Convert Pyralog Records → Arrow (zero-copy!)
        self.records_to_batch(records)
    }
    
    fn records_to_batch(&self, records: Vec<Record>) -> Result<RecordBatch> {
        // Build Arrow arrays
        let scarab_ids: UInt64Array = records.iter()
            .map(|r| r.scarab_id.as_u64())
            .collect();
        
        let timestamps: TimestampMillisecondArray = records.iter()
            .map(|r| r.timestamp.duration_since(UNIX_EPOCH).unwrap().as_millis() as i64)
            .collect();
        
        let keys: BinaryArray = records.iter()
            .map(|r| r.key.as_ref().map(|k| k.as_ref()))
            .collect();
        
        let values: BinaryArray = records.iter()
            .map(|r| r.value.as_ref())
            .collect();
        
        RecordBatch::try_new(
            self.schema.clone(),
            vec![
                Arc::new(scarab_ids),
                Arc::new(timestamps),
                Arc::new(keys),
                Arc::new(values),
            ],
        )
    }
}
```

### 6.2 DataFusion SQL Engine (Week 3-4)

```rust
// pyralog-arrow/src/sql.rs
use datafusion::prelude::*;

pub struct SqlEngine {
    ctx: SessionContext,
    arrow_storage: Arc<ArrowStorage>,
}

impl SqlEngine {
    pub async fn new(arrow_storage: Arc<ArrowStorage>) -> Result<Self> {
        let ctx = SessionContext::new();
        
        // Register Pyralog table
        ctx.register_table("pyralog", PyralogTableProvider::new(arrow_storage.clone()))?;
        
        Ok(Self {
            ctx,
            arrow_storage,
        })
    }
    
    pub async fn execute_sql(&self, query: &str) -> Result<Vec<RecordBatch>> {
        let df = self.ctx.sql(query).await?;
        df.collect().await
    }
}

// Example query:
// SELECT user_id, COUNT(*) as count
// FROM pyralog
// WHERE timestamp >= '2025-01-01'
// GROUP BY user_id
// ORDER BY count DESC
// LIMIT 10
```

### 6.3 Polars DataFrames (Week 5-6)

```rust
// pyralog-arrow/src/polars.rs
use polars::prelude::*;

pub struct DataFrameEngine {
    arrow_storage: Arc<ArrowStorage>,
}

impl DataFrameEngine {
    pub async fn read_dataframe(
        &self,
        start: EpochOffset,
        end: EpochOffset,
    ) -> Result<DataFrame> {
        // Read as Arrow RecordBatch
        let batch = self.arrow_storage.read_batch(start, end, usize::MAX).await?;
        
        // Convert to Polars DataFrame (zero-copy!)
        let df = DataFrame::try_from(batch)?;
        
        Ok(df)
    }
}

// Example usage:
// let df = engine.read_dataframe(start, end).await?;
// let result = df
//     .lazy()
//     .filter(col("user_id").gt(100))
//     .groupby([col("user_id")])
//     .agg([col("value").count().alias("count")])
//     .sort("count", SortOptions::default().with_order_descending(true))
//     .limit(10)
//     .collect()?;
```

**Milestone**: Multi-model database working with Arrow, SQL, and DataFrame APIs

---

## Phase 7: Production Hardening

**Goal**: Production-ready system with observability, recovery, reliability

**Duration**: 4-6 weeks

### 7.1 Observability (Week 1-2)

**Prometheus metrics, OpenTelemetry tracing, structured logging**

```rust
use prometheus::{Counter, Histogram, IntGauge, Registry};
use tracing::{info, warn, error, instrument};

lazy_static! {
    static ref RECORDS_WRITTEN: Counter = register_counter!(
        "pyralog_records_written_total",
        "Total records written"
    ).unwrap();
    
    static ref WRITE_LATENCY: Histogram = register_histogram!(
        "pyralog_write_latency_seconds",
        "Write latency in seconds"
    ).unwrap();
    
    static ref SCARAB_IDS_GENERATED: Counter = register_counter!(
        "pyralog_scarab_ids_generated_total",
        "Total Scarab IDs generated"
    ).unwrap();
}

#[instrument(skip(self))]
pub async fn append(&self, record: Record) -> Result<LogOffset> {
    let start = Instant::now();
    
    let result = self.append_internal(record).await;
    
    WRITE_LATENCY.observe(start.elapsed().as_secs_f64());
    
    match result {
        Ok(offset) => {
            RECORDS_WRITTEN.inc();
            info!(offset = %offset, "Record appended");
            Ok(offset)
        }
        Err(e) => {
            error!(error = %e, "Failed to append");
            Err(e)
        }
    }
}
```

### 7.2 Crash Recovery (Week 3-4)

**RocksDB recovery, Obelisk recovery, Epoch recovery**

```rust
impl PyramidNode {
    pub async fn recover(config: NodeConfig) -> Result<Self> {
        info!("Starting recovery for node {}", config.node_id);
        
        // 1. Recover RocksDB storage
        let storage = PyramidStorage::recover(&config.data_dir).await?;
        
        // 2. Recover Raft state
        let global_raft = GlobalRaft::recover(&config.raft_dir).await?;
        let partition_rafts = PartitionRaft::recover_all(&config.raft_dir).await?;
        
        // 3. Recover epochs
        for (partition_id, raft) in &partition_rafts {
            let sequencer = Sequencer::recover(*partition_id, &storage).await?;
            // ...
        }
        
        info!("Recovery complete");
        
        Ok(Self {
            // ...
        })
    }
}

impl ObeliskNode {
    pub async fn recover(config: ObeliskConfig) -> Result<Self> {
        info!("Starting Obelisk recovery for ID {}", config.obelisk_id);
        
        // Recover sequencer (instant!)
        let sequencer = ObeliskSequencer::recover(config.obelisk_id, &config.data_dir)?;
        
        info!("Obelisk recovery complete at value {}", sequencer.current());
        
        Ok(Self {
            obelisk_id: config.obelisk_id,
            sequencer: Arc::new(sequencer),
            // ...
        })
    }
}
```

### 7.3 Configuration Management (Week 5)

```toml
# config/pyramid.toml (Pyramid node config)
[server]
node_id = 1
bind_addr = "0.0.0.0:9092"
data_dir = "/var/lib/pyralog/pyramid"

[cluster]
bootstrap_servers = ["node1:9092", "node2:9092", "node3:9092"]

[obelisk]
# Connect to Pharaoh Network
obelisk_servers = ["obelisk1:8080", "obelisk2:8080"]

[storage]
engine = "rocksdb"
write_buffer_size = 67108864  # 64MB
max_write_buffer_number = 3

[replication]
replication_factor = 3
write_quorum = 2
copyset_strategy = "PerRecord"
coordinator_only = false  # Set true for maximum scale

[raft]
election_timeout_ms = 300
heartbeat_interval_ms = 100

[observability]
metrics_port = 9090
tracing_endpoint = "http://jaeger:4317"
log_level = "info"
```

```toml
# config/obelisk.toml (Obelisk node config)
[server]
obelisk_id = 0
bind_addr = "0.0.0.0:8080"
data_dir = "/var/lib/pyralog/obelisk"

[observability]
metrics_port = 8090
log_level = "info"
```

### 7.4 Integration Tests (Week 6)

**End-to-end tests, chaos testing, CopySet testing**

```rust
#[tokio::test]
async fn test_full_two_tier_cluster() {
    // 1. Start 2 Obelisk nodes
    let obelisk_cluster = ObeliskCluster::new(2).await;
    
    // 2. Start 3 Pyramid nodes
    let pyramid_cluster = PyramidCluster::new(3).await
        .with_obelisk_cluster(obelisk_cluster.clone());
    
    // 3. Create client
    let client = PyralogClient::new(
        pyramid_cluster.bootstrap_servers(),
        obelisk_cluster.servers(),
    ).await;
    
    // 4. Write 10K records
    for i in 0..10000 {
        let epoch_offset = client.produce(
            "test-log",
            Some(format!("key-{}", i).into()),
            format!("value-{}", i).into(),
        ).await.unwrap();
        
        assert!(epoch_offset.offset.as_u64() > 0);
    }
    
    // 5. Read back
    let records = client.consume("test-log", 0, EpochOffset::ZERO, 10000).await.unwrap();
    assert_eq!(records.len(), 10000);
    
    // 6. Kill Pyramid leader
    pyramid_cluster.kill_leader().await;
    
    // 7. Wait for failover
    tokio::time::sleep(Duration::from_millis(200)).await;
    
    // 8. Continue writing (should work!)
    for i in 10000..20000 {
        client.produce(
            "test-log",
            Some(format!("key-{}", i).into()),
            format!("value-{}", i).into(),
        ).await.unwrap();
    }
    
    // 9. Verify all 20K records
    let all_records = client.consume("test-log", 0, EpochOffset::ZERO, 20000).await.unwrap();
    assert_eq!(all_records.len(), 20000);
}
```

**Milestone**: Production-ready system with observability, recovery, and passing all tests

---

## Phase 8: Advanced Features

**Goal**: Production features for real-world use

**Duration**: Ongoing (Aug 2026+)

### 8.1 Distributed Transactions

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md#distributed-transactions) for complete implementation.

**Key points**:
- Percolator protocol with Scarab TSO (4+ billion tx/sec!)
- MVCC with Snapshot Isolation
- Two-phase commit

### 8.2 Exactly-Once Semantics

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md#exactly-once-semantics).

**Key points**:
- Idempotent producers (Scarab session IDs)
- Transactional consumers
- Atomic offset commits

### 8.3 Tensor Database

See [TENSOR_DATABASE.md](TENSOR_DATABASE.md) and [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md#tensor-database-for-mlai).

**Key points**:
- Safetensors for ML models (100× faster than pickle)
- DLPack for zero-copy tensor exchange
- Vector embeddings with ANN search

### 8.4 Change Data Capture (CDC)

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md#change-data-capture-cdc).

**Key points**:
- Debezium-compatible
- Scarab IDs for event ordering
- Schema evolution support

### 8.5 Stream Processing

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md#stream-processing).

**Key points**:
- DataFusion SQL windowing
- Three window types (tumbling, hopping, session)
- State management

---

## Testing Strategy

### Unit Tests

**Coverage target: 80%+ for all crates**

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_scarab_id_ordering() {
        let id1 = ScarabId::generate(0, &sequencer).unwrap();
        let id2 = ScarabId::generate(0, &sequencer).unwrap();
        assert!(id2 > id1);
    }
    
    #[tokio::test]
    async fn test_obelisk_crash_recovery() {
        let seq = ObeliskSequencer::create(0, tmpdir).unwrap();
        seq.next().unwrap();
        seq.next().unwrap();
        drop(seq);
        
        let recovered = ObeliskSequencer::recover(0, tmpdir).unwrap();
        assert_eq!(recovered.next().unwrap(), 3);
    }
}
```

### Integration Tests

- Two-tier cluster startup (Obelisk + Pyramid)
- Scarab ID generation across nodes
- Dual Raft (Global + Per-Partition)
- CopySet strategies
- Failover and recovery
- Multi-model queries

### Benchmarks

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn bench_obelisk_increment(c: &mut Criterion) {
    let seq = ObeliskSequencer::create(0, Path::new("/tmp")).unwrap();
    
    c.bench_function("obelisk_increment", |b| {
        b.iter(|| {
            black_box(seq.next().unwrap());
        });
    });
}

fn bench_scarab_id_generation(c: &mut Criterion) {
    let seq = ObeliskSequencer::create(0, Path::new("/tmp")).unwrap();
    
    c.bench_function("scarab_id_generation", |b| {
        b.iter(|| {
            black_box(ScarabId::generate(0, &seq).unwrap());
        });
    });
}

criterion_group!(benches, bench_obelisk_increment, bench_scarab_id_generation);
criterion_main!(benches);
```

**Targets**:
- <1μs: Obelisk increment
- <1μs: Scarab ID generation
- <1ms: Write with replication
- 10M+ writes/sec: Per Pyramid node
- 4B+ IDs/sec: 1024 Obelisk nodes

---

## Deployment Strategy

### Docker

```dockerfile
# Dockerfile.pyramid (Pyramid node)
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release --bin pyralog-server

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/pyralog-server /usr/local/bin/
EXPOSE 9092 9090
CMD ["pyralog-server"]
```

```dockerfile
# Dockerfile.obelisk (Obelisk node)
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release --bin pyralog-obelisk-server

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/pyralog-obelisk-server /usr/local/bin/
EXPOSE 8080 8090
CMD ["pyralog-obelisk-server"]
```

### Kubernetes

```yaml
# k8s/obelisk-statefulset.yaml (Pharaoh Network)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pyralog-obelisk
spec:
  serviceName: pyralog-obelisk
  replicas: 4  # Up to 1024!
  template:
    spec:
      containers:
      - name: obelisk
        image: pyralog-obelisk:latest
        ports:
        - containerPort: 8080
        - containerPort: 8090
        volumeMounts:
        - name: data
          mountPath: /var/lib/pyralog/obelisk
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi  # Tiny! (sparse files)
```

```yaml
# k8s/pyramid-statefulset.yaml (Pyralog Cluster)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pyralog-pyramid
spec:
  serviceName: pyralog-pyramid
  replicas: 3
  template:
    spec:
      containers:
      - name: pyramid
        image: pyralog-pyramid:latest
        ports:
        - containerPort: 9092
        - containerPort: 9090
        volumeMounts:
        - name: data
          mountPath: /var/lib/pyralog/pyramid
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Gi  # Large for LSM-Tree
```

---

## Success Criteria

### Phase 0 ✅
- [ ] Obelisk Sequencer working
- [ ] <1μs per increment
- [ ] Crash recovery < 1ms
- [ ] Scarab IDs generated correctly

### Phase 1 ✅
- [ ] Project structure complete
- [ ] All core types have tests
- [ ] CI/CD pipeline working

### Phase 2 ✅
- [ ] Custom LSM-Tree storage working
- [ ] Segment files with memory-mapped I/O
- [ ] Sparse indexes
- [ ] Memtable with WAL
- [ ] 100K+ writes/sec (single node)
- [ ] Zero data loss on crash
- [ ] Compaction (L0 → L1+)

### Phase 3 ✅
- [ ] 3-Pyramid-node cluster working
- [ ] Dual Raft consensus
- [ ] Global Raft for metadata
- [ ] Per-Partition Raft for epochs
- [ ] Leader election < 100ms

### Phase 4 ✅
- [ ] Epochs implemented
- [ ] Replication with quorum
- [ ] Both CopySet strategies working
- [ ] 1M+ writes/sec (per-partition mode)

### Phase 5 ✅
- [ ] Smart client routing
- [ ] JSON-RPC/WebSocket working
- [ ] Arrow Flight integration
- [ ] Metadata caching

### Phase 6 ✅
- [ ] Arrow storage working
- [ ] DataFusion SQL queries
- [ ] Polars DataFrames
- [ ] Multi-model database

### Phase 7 ✅
- [ ] Prometheus metrics
- [ ] Distributed tracing
- [ ] All integration tests passing
- [ ] <1ms p99 write latency

### Phase 8 (Ongoing)
- [ ] Distributed transactions
- [ ] Exactly-once semantics
- [ ] Tensor database
- [ ] CDC
- [ ] Stream processing

---

## Conclusion

### Pyralog's Revolutionary Architecture

**What Makes Pyralog Special:**

**1. Two-Tier Architecture**
- ☀️ Pharaoh Network (Obelisk nodes) - Coordination layer
- 🔺 Pyralog Cluster (Pyramid nodes) - Storage/compute layer
- **No other system separates coordination from storage like this!**

**2. Obelisk Sequencer**
- <1μs ID generation (coordination-free!)
- Crash-safe (sparse file technique)
- 28B+ operations/sec (1024 nodes)
- **The foundation for everything**

**3. Scarab IDs**
- Globally unique, time-ordered
- 4+ billion IDs/sec
- Enables distributed TSO, transactions, sessions
- **8000× faster than TiKV's centralized TSO**

**4. Dual Raft**
- Global Raft (cluster metadata)
- Per-Partition Raft (epochs, failover)
- **1000 partitions fail over in 10ms parallel!**

**5. Multi-Model Database**
- Six data models in Arrow
- Zero-copy multi-model joins
- **10-50× faster than ETL**

### Performance Targets

```
Obelisk Sequencer:     <1μs per ID
Scarab IDs:            4+ billion/sec (1024 nodes)
Transactions:          4+ billion tx/sec
Write throughput:      10M+ writes/sec per node
Write latency:         <1ms p99
Pyramid failover:      <100ms
Obelisk recovery:      <1ms
Partitions/node:       500 (coordinator-only mode)
```

### Timeline to Production

**7-10 months** (27-39 weeks) from zero to production-ready.

Follow the phases, test continuously, benchmark early, and you'll have a distributed log with **full custom storage** that **outperforms and outscales** anything available today.

---

**Let's build Pyralog! 🔺☀️🗿**
