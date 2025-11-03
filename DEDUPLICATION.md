# Deduplication in Pyralog

**Comprehensive guide to deduplication strategies across all system levels**

---

## Table of Contents

1. [Overview](#overview)
2. [Deduplication Layers](#deduplication-layers)
3. [Storage-Level Deduplication](#storage-level-deduplication)
4. [PPHM Deduplication](#pphm-deduplication)
5. [Exactly-Once Semantics](#exactly-once-semantics)
6. [Content-Addressable Storage](#content-addressable-storage)
7. [Application-Level Patterns](#application-level-patterns)
8. [Performance Considerations](#performance-considerations)
9. [Best Practices](#best-practices)

---

## Overview

Pyralog implements **multi-layer deduplication** to ensure data efficiency, correctness, and performance across the entire stack. Deduplication happens at different levels, each serving a specific purpose:

```
┌─────────────────────────────────────────────────────────┐
│                   Application Layer                      │
│           (Business Logic Deduplication)                 │
├─────────────────────────────────────────────────────────┤
│                  Exactly-Once Layer                      │
│        (Session-Based Write Deduplication)               │
├─────────────────────────────────────────────────────────┤
│                    PPHM Layer                            │
│        (Perfect Hash Map Merge Deduplication)            │
├─────────────────────────────────────────────────────────┤
│                  Storage Layer                           │
│           (LSM Compaction Deduplication)                 │
├─────────────────────────────────────────────────────────┤
│              Content-Addressable Layer                   │
│          (Chunk-Level Hash Deduplication)                │
└─────────────────────────────────────────────────────────┘
```

---

## Deduplication Layers

### Layer Comparison

| Layer | Scope | Granularity | Cost | When to Use |
|-------|-------|-------------|------|-------------|
| **Application** | Business logic | Record/object | Low | Semantic deduplication, custom rules |
| **Exactly-Once** | Write path | Write request | Medium | Idempotent writes, retry safety |
| **PPHM** | Index merging | Key-value pairs | Medium | Compacting indexes, routing tables |
| **LSM Compaction** | Storage | Log segments | High | Storage reclamation, version merging |
| **Content-Addressable** | Block storage | Data chunks | Very High | Backup, archival, immutable data |

### Why Multiple Layers?

Each layer serves a different purpose:

1. **Application**: Business logic (e.g., "same user email")
2. **Exactly-Once**: Correctness (e.g., "same write request")
3. **PPHM**: Index efficiency (e.g., "same key in multiple indexes")
4. **LSM**: Storage efficiency (e.g., "same key across segments")
5. **Content-Addressable**: Block-level efficiency (e.g., "same chunk in multiple files")

---

## Storage-Level Deduplication

### LSM Compaction

**Purpose**: Merge multiple log segments and deduplicate keys across levels

#### Compaction Strategies

##### 1. **Last-Writer-Wins (LWW)**

```rust
/// Default: Keep the most recent version
pub struct LWWCompactionStrategy;

impl CompactionStrategy for LWWCompactionStrategy {
    fn merge_records(&self, key: &[u8], records: Vec<Record>) -> Record {
        // Records sorted by (LSN, timestamp) descending
        records.into_iter()
            .max_by_key(|r| (r.lsn, r.timestamp))
            .expect("at least one record")
    }
}
```

**Use case**: Event logs, time-series data, mutable state

**Example**:
```
Input segments:
  Segment 1: user:123 → {name: "Alice", age: 25} @ LSN=100
  Segment 2: user:123 → {name: "Alice", age: 26} @ LSN=200
  Segment 3: user:123 → {name: "Alicia", age: 27} @ LSN=300

Output (compacted):
  user:123 → {name: "Alicia", age: 27} @ LSN=300  ← Kept (highest LSN)
```

##### 2. **Tombstone-Based Deletion**

```rust
/// Handle deletion markers
pub struct TombstoneStrategy {
    tombstone_retention: Duration,  // e.g., 7 days
}

impl CompactionStrategy for TombstoneStrategy {
    fn merge_records(&self, key: &[u8], mut records: Vec<Record>) -> Option<Record> {
        // Sort by timestamp descending
        records.sort_by_key(|r| std::cmp::Reverse(r.timestamp));
        
        let latest = &records[0];
        
        // If latest is tombstone
        if latest.is_tombstone() {
            let tombstone_age = Instant::now() - latest.timestamp;
            
            // Keep tombstone if within retention period
            if tombstone_age < self.tombstone_retention {
                return Some(latest.clone());
            } else {
                // Tombstone expired - remove key entirely
                return None;
            }
        }
        
        // Keep latest non-tombstone
        Some(latest.clone())
    }
}
```

**Use case**: Deletions, GDPR right-to-be-forgotten, data expiration

**Example**:
```
Input segments:
  Segment 1: user:123 → {name: "Alice"} @ T=0
  Segment 2: user:123 → [TOMBSTONE] @ T=100
  
Output (within retention):
  user:123 → [TOMBSTONE] @ T=100  ← Kept (tombstone still valid)
  
Output (after retention):
  (key removed entirely)
```

##### 3. **MVCC Compaction**

```rust
/// Multi-Version Concurrency Control: Keep multiple versions
pub struct MVCCStrategy {
    max_versions: usize,  // e.g., 5
    min_age: Duration,    // e.g., 30 days
}

impl CompactionStrategy for MVCCStrategy {
    fn merge_records(&self, key: &[u8], mut records: Vec<Record>) -> Vec<Record> {
        // Sort by LSN descending
        records.sort_by_key(|r| std::cmp::Reverse(r.lsn));
        
        // Keep up to max_versions
        let mut kept = Vec::new();
        
        for record in records {
            // Always keep recent versions
            if kept.len() < self.max_versions {
                kept.push(record);
                continue;
            }
            
            // Keep old versions if still within min_age
            let age = Instant::now() - record.timestamp;
            if age < self.min_age {
                kept.push(record);
            }
        }
        
        kept
    }
}
```

**Use case**: Time-travel queries, audit trails, immutable knowledge base

**Example**:
```
Input segments (max_versions=3):
  user:123 @ LSN=100: {age: 25}
  user:123 @ LSN=200: {age: 26}
  user:123 @ LSN=300: {age: 27}
  user:123 @ LSN=400: {age: 28}
  user:123 @ LSN=500: {age: 29}

Output (compacted):
  user:123 @ LSN=500: {age: 29}  ← Kept
  user:123 @ LSN=400: {age: 28}  ← Kept
  user:123 @ LSN=300: {age: 27}  ← Kept
  (older versions removed)
```

##### 4. **Delta Encoding**

```rust
/// Store only changes between versions
pub struct DeltaEncodingStrategy;

impl CompactionStrategy for DeltaEncodingStrategy {
    fn merge_records(&self, key: &[u8], mut records: Vec<Record>) -> Vec<Record> {
        records.sort_by_key(|r| r.lsn);
        
        let mut result = Vec::new();
        let mut base = records[0].clone();
        result.push(base.clone());
        
        for record in &records[1..] {
            // Compute delta from base
            let delta = compute_delta(&base, record);
            
            if delta.size() < record.size() / 2 {
                // Delta is significantly smaller - store delta
                result.push(Record::delta(delta));
            } else {
                // Delta not worth it - store full record and update base
                result.push(record.clone());
                base = record.clone();
            }
        }
        
        result
    }
}
```

**Use case**: Slowly-changing dimensions, configuration history, schema evolution

---

## PPHM Deduplication

Perfect Hash Map merging requires resolving duplicate keys across multiple input maps. See [PPHM.md](PPHM.md) for full details.

### Built-in Strategies

#### 1. **Last-Writer-Wins (LWW)**

```rust
pub struct LastWriterWins;

impl<K, V> Reducer<K, V> for LastWriterWins {
    fn reduce(&self, _key: &K, values: &[V]) -> Result<V> {
        Ok(values.last().unwrap().clone())
    }
}
```

**Use case**: Default strategy, simple semantics

#### 2. **First-Wins**

```rust
pub struct FirstWins;

impl<K, V> Reducer<K, V> for FirstWins {
    fn reduce(&self, _key: &K, values: &[V]) -> Result<V> {
        Ok(values.first().unwrap().clone())
    }
}
```

**Use case**: Prefer oldest data, historical priority

#### 3. **Max/Min Value**

```rust
pub struct MaxValue;

impl<K, V: Ord> Reducer<K, V> for MaxValue {
    fn reduce(&self, _key: &K, values: &[V]) -> Result<V> {
        Ok(values.iter().max().unwrap().clone())
    }
}

pub struct MinValue;

impl<K, V: Ord> Reducer<K, V> for MinValue {
    fn reduce(&self, _key: &K, values: &[V]) -> Result<V> {
        Ok(values.iter().min().unwrap().clone())
    }
}
```

**Use case**: Numerical aggregation, monotonic counters

#### 4. **Priority-Based**

```rust
pub struct PriorityReducer {
    /// Priority per input source (higher = keep)
    priorities: Vec<u8>,
}

impl<K, V> Reducer<K, (usize, V)> for PriorityReducer {
    fn reduce(&self, _key: &K, values: &[(usize, V)]) -> Result<V> {
        values.iter()
            .max_by_key(|(source_id, _)| self.priorities[*source_id])
            .map(|(_, v)| v.clone())
            .ok_or_else(|| Error::EmptyValues)
    }
}
```

**Use case**: Configuration hierarchies, ACL merging

**Example**:
```rust
// Config hierarchy: default < env < file < cli
let reducer = PriorityReducer {
    priorities: vec![
        1,  // default.toml
        2,  // environment variables
        3,  // config.toml
        4,  // CLI overrides (highest)
    ],
};

let config = merge_configs(&[
    default_config,  // source_id=0
    env_config,      // source_id=1
    file_config,     // source_id=2
    cli_config,      // source_id=3
], reducer);
```

#### 5. **Timestamp-Based (CRDT)**

```rust
pub struct TimestampReducer;

#[derive(Clone)]
pub struct Timestamped<V> {
    value: V,
    timestamp: u64,    // Lamport timestamp
    node_id: u64,      // Tie-breaker
}

impl<K, V: Clone> Reducer<K, Timestamped<V>> for TimestampReducer {
    fn reduce(&self, _key: &K, values: &[Timestamped<V>]) -> Result<Timestamped<V>> {
        Ok(values.iter()
            .max_by_key(|tv| (tv.timestamp, tv.node_id))
            .unwrap()
            .clone())
    }
}
```

**Use case**: Distributed systems, eventual consistency, CRDTs

#### 6. **Custom Merge (Deep Merge, Operational Transform)**

```rust
pub struct JsonMerger;

impl<K> Reducer<K, JsonValue> for JsonMerger {
    fn reduce(&self, _key: &K, values: &[JsonValue]) -> Result<JsonValue> {
        let mut result = json!({});
        
        // Deep merge all objects
        for value in values {
            deep_merge(&mut result, value);
        }
        
        Ok(result)
    }
}

fn deep_merge(target: &mut JsonValue, source: &JsonValue) {
    if let (Some(target_obj), Some(source_obj)) = (
        target.as_object_mut(),
        source.as_object(),
    ) {
        for (key, value) in source_obj {
            if let Some(target_value) = target_obj.get_mut(key) {
                // Recursively merge nested objects
                deep_merge(target_value, value);
            } else {
                // Insert new key
                target_obj.insert(key.clone(), value.clone());
            }
        }
    } else {
        // Not objects - replace
        *target = source.clone();
    }
}
```

**Use case**: JSON documents, nested structures, schema evolution

**Example**:
```rust
// Input maps:
// Map 1: {"user": {"name": "Alice", "age": 25}}
// Map 2: {"user": {"age": 26, "city": "NYC"}}
// Map 3: {"admin": true}

// Output (deep merged):
// {"user": {"name": "Alice", "age": 26, "city": "NYC"}, "admin": true}
```

---

## Exactly-Once Semantics

**Purpose**: Deduplicate write requests during retries and failures

### Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         Producer                              │
│  ┌────────────────────────────────────────────────────┐      │
│  │ session_id = obelisk.next()  // Crash-safe        │      │
│  │ epoch = current_epoch                               │      │
│  │ sequence = 0                                        │      │
│  └────────────────────────────────────────────────────┘      │
│                           │                                   │
│                           ▼                                   │
│          ┌─────────────────────────────────┐                 │
│          │  Write: (session_id, epoch, seq) │                │
│          └─────────────────────────────────┘                 │
└─────────────────────────────┬────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                       Pyralog Broker                          │
│  ┌────────────────────────────────────────────────────┐      │
│  │         Deduplication Cache (per-partition)        │      │
│  │  Key: (session_id, partition_id)                   │      │
│  │  Value: (last_epoch, last_sequence, last_offset)   │      │
│  └────────────────────────────────────────────────────┘      │
│                           │                                   │
│                           ▼                                   │
│              ┌───────────────────────┐                        │
│              │  Check: Is duplicate?  │                       │
│              └───────────────────────┘                        │
│                    │            │                             │
│                 Yes│            │No                           │
│                    │            │                             │
│            ┌───────▼─┐      ┌──▼──────┐                      │
│            │ Return  │      │ Append  │                      │
│            │ cached  │      │ to log  │                      │
│            │ offset  │      │         │                      │
│            └─────────┘      └─────────┘                      │
└──────────────────────────────────────────────────────────────┘
```

### Implementation

```rust
pub struct DeduplicationCache {
    /// Key: (session_id, partition_id)
    /// Value: (last_epoch, last_sequence, last_offset)
    cache: RwLock<LruCache<(SessionId, u32), (Epoch, u32, LogOffset)>>,
    capacity: usize,
}

impl DeduplicationCache {
    pub fn check_duplicate(
        &self,
        session_id: SessionId,
        partition_id: u32,
        epoch: Epoch,
        sequence: u32,
    ) -> Option<LogOffset> {
        let cache = self.cache.read().unwrap();
        
        if let Some((cached_epoch, cached_seq, cached_offset)) = 
            cache.get(&(session_id, partition_id)) 
        {
            // Same epoch and sequence - exact duplicate
            if *cached_epoch == epoch && *cached_seq == sequence {
                return Some(*cached_offset);
            }
            
            // Old epoch - producer restarted, allow write
            if epoch > *cached_epoch {
                return None;
            }
            
            // Same epoch, old sequence - out of order replay, reject
            if epoch == *cached_epoch && sequence <= *cached_seq {
                return Some(*cached_offset);
            }
        }
        
        None
    }
    
    pub fn record_write(
        &self,
        session_id: SessionId,
        partition_id: u32,
        epoch: Epoch,
        sequence: u32,
        offset: LogOffset,
    ) {
        let mut cache = self.cache.write().unwrap();
        cache.insert(
            (session_id, partition_id),
            (epoch, sequence, offset),
        );
    }
}
```

### Producer Implementation

```rust
pub struct PyralogProducer {
    session_id: SessionId,
    epoch: Epoch,
    sequence: AtomicU32,
    obelisk: Arc<ObeliskSequencer>,
}

impl PyralogProducer {
    pub async fn new(obelisk: Arc<ObeliskSequencer>) -> Result<Self> {
        // Generate unique session ID (crash-safe)
        let session_id = obelisk.next().await?;
        
        Ok(Self {
            session_id: SessionId(session_id),
            epoch: Epoch(0),
            sequence: AtomicU32::new(0),
            obelisk,
        })
    }
    
    pub async fn send(&self, record: Record) -> Result<LogOffset> {
        let seq = self.sequence.fetch_add(1, Ordering::SeqCst);
        
        let request = ProduceRequest {
            session_id: self.session_id,
            epoch: self.epoch,
            sequence: seq,
            record,
        };
        
        // Send with retries
        let mut retries = 0;
        loop {
            match self.client.produce(request.clone()).await {
                Ok(offset) => return Ok(offset),
                Err(e) if e.is_retriable() && retries < MAX_RETRIES => {
                    retries += 1;
                    tokio::time::sleep(RETRY_BACKOFF).await;
                }
                Err(e) => return Err(e),
            }
        }
    }
}
```

### Benefits

✅ **Idempotent writes**: Safe to retry without creating duplicates  
✅ **Exactly-once delivery**: Guaranteed across failures  
✅ **No application logic**: Deduplication handled by infrastructure  
✅ **Crash-safe**: Session IDs survive producer restarts via Obelisk  
✅ **Low overhead**: O(1) cache lookup per write

---

## Content-Addressable Storage

**Purpose**: Deduplicate identical data chunks across files (backup, archival)

### Architecture

```rust
pub struct ContentAddressableStore {
    /// Key: BLAKE3 hash of chunk
    /// Value: Reference count + storage location
    chunks: Arc<RwLock<HashMap<Blake3Hash, ChunkMetadata>>>,
    storage: Box<dyn BlobStorage>,
}

#[derive(Clone)]
pub struct ChunkMetadata {
    hash: Blake3Hash,
    size: usize,
    refcount: Arc<AtomicU64>,
    storage_path: PathBuf,
}

impl ContentAddressableStore {
    /// Store data with deduplication
    pub async fn store(&self, data: &[u8]) -> Result<ContentHandle> {
        // Split into chunks (e.g., 4MB)
        let chunks = self.chunk(data);
        
        let mut chunk_handles = Vec::new();
        
        for chunk in chunks {
            // Hash chunk
            let hash = blake3::hash(chunk);
            
            // Check if already exists
            let mut chunks_map = self.chunks.write().unwrap();
            
            if let Some(metadata) = chunks_map.get_mut(&hash) {
                // Chunk exists - increment refcount
                metadata.refcount.fetch_add(1, Ordering::SeqCst);
                chunk_handles.push(hash);
            } else {
                // New chunk - store it
                let path = self.storage.write(hash, chunk).await?;
                
                let metadata = ChunkMetadata {
                    hash,
                    size: chunk.len(),
                    refcount: Arc::new(AtomicU64::new(1)),
                    storage_path: path,
                };
                
                chunks_map.insert(hash, metadata);
                chunk_handles.push(hash);
            }
        }
        
        Ok(ContentHandle { chunks: chunk_handles })
    }
    
    /// Retrieve data
    pub async fn retrieve(&self, handle: &ContentHandle) -> Result<Vec<u8>> {
        let chunks_map = self.chunks.read().unwrap();
        
        let mut data = Vec::new();
        
        for hash in &handle.chunks {
            let metadata = chunks_map.get(hash)
                .ok_or_else(|| Error::ChunkNotFound(*hash))?;
            
            let chunk = self.storage.read(&metadata.storage_path).await?;
            data.extend_from_slice(&chunk);
        }
        
        Ok(data)
    }
    
    /// Delete data (decrement refcounts)
    pub async fn delete(&self, handle: ContentHandle) -> Result<()> {
        let mut chunks_map = self.chunks.write().unwrap();
        
        for hash in handle.chunks {
            if let Some(metadata) = chunks_map.get(&hash) {
                let refcount = metadata.refcount.fetch_sub(1, Ordering::SeqCst);
                
                // Last reference - delete chunk
                if refcount == 1 {
                    self.storage.delete(&metadata.storage_path).await?;
                    chunks_map.remove(&hash);
                }
            }
        }
        
        Ok(())
    }
}
```

### Chunking Strategies

#### 1. **Fixed-Size Chunking**

```rust
fn fixed_size_chunking(data: &[u8], chunk_size: usize) -> Vec<&[u8]> {
    data.chunks(chunk_size).collect()
}
```

**Pros**: Simple, predictable  
**Cons**: Poor deduplication if data shifts

#### 2. **Content-Defined Chunking (CDC)**

```rust
use fastcdc::FastCDC;

fn content_defined_chunking(
    data: &[u8],
    avg_size: usize,
    min_size: usize,
    max_size: usize,
) -> Vec<&[u8]> {
    let cdc = FastCDC::new(data, min_size, avg_size, max_size);
    
    cdc.map(|chunk| {
        &data[chunk.offset..chunk.offset + chunk.length]
    }).collect()
}
```

**Pros**: Better deduplication (finds natural boundaries)  
**Cons**: Variable chunk sizes, higher CPU cost

### Use Cases

1. **Backup & Restore**: Incremental backups with deduplication
2. **Tiered Storage**: Deduplicate cold data in S3
3. **Snapshot Storage**: Space-efficient snapshots
4. **Data Archival**: Long-term storage with compression + dedup

### Deduplication Ratios

| Data Type | Fixed-Size | CDC | Compression + CDC |
|-----------|-----------|-----|-------------------|
| **Virtual Machine Images** | 2-5× | 10-20× | 20-50× |
| **Database Backups** | 3-8× | 15-30× | 30-100× |
| **Log Files** | 2-4× | 5-10× | 20-50× |
| **Source Code** | 5-15× | 20-50× | 50-200× |
| **Random Data** | 1× | 1× | 1× |

---

## Application-Level Patterns

### 1. **Semantic Deduplication**

```rust
/// Business logic deduplication (e.g., "same email = same user")
pub trait SemanticDeduplicator<T> {
    fn canonical_key(&self, item: &T) -> Vec<u8>;
    fn should_merge(&self, existing: &T, new: &T) -> bool;
    fn merge(&self, existing: T, new: T) -> T;
}

/// Example: User email deduplication
pub struct UserDeduplicator;

impl SemanticDeduplicator<User> for UserDeduplicator {
    fn canonical_key(&self, user: &User) -> Vec<u8> {
        // Normalize email: lowercase, trim
        user.email.trim().to_lowercase().into_bytes()
    }
    
    fn should_merge(&self, existing: &User, new: &User) -> bool {
        // Merge if emails match (case-insensitive)
        self.canonical_key(existing) == self.canonical_key(new)
    }
    
    fn merge(&self, mut existing: User, new: User) -> User {
        // Keep newer data for most fields
        if new.updated_at > existing.updated_at {
            existing.name = new.name;
            existing.age = new.age;
            existing.updated_at = new.updated_at;
        }
        
        // Merge arrays (union)
        existing.tags.extend(new.tags);
        existing.tags.sort();
        existing.tags.dedup();
        
        existing
    }
}
```

### 2. **Sliding Window Deduplication**

```rust
/// Deduplicate within a time window
pub struct SlidingWindowDeduplicator<K> {
    seen: RwLock<HashMap<K, Instant>>,
    window: Duration,
}

impl<K: Hash + Eq> SlidingWindowDeduplicator<K> {
    pub fn check_and_mark(&self, key: K) -> bool {
        let now = Instant::now();
        let mut seen = self.seen.write().unwrap();
        
        // Clean expired entries
        seen.retain(|_, timestamp| now.duration_since(*timestamp) < self.window);
        
        // Check if seen
        if seen.contains_key(&key) {
            return true; // Duplicate
        }
        
        // Mark as seen
        seen.insert(key, now);
        false
    }
}
```

**Use case**: Duplicate event detection, idempotency tokens, rate limiting

### 3. **Bloom Filter Deduplication**

```rust
use probabilistic_collections::bloom::BloomFilter;

/// Space-efficient approximate deduplication
pub struct BloomDeduplicator {
    filter: RwLock<BloomFilter>,
    fpr: f64,  // False positive rate (e.g., 0.01)
}

impl BloomDeduplicator {
    pub fn check_and_mark(&self, key: &[u8]) -> Result<bool, ()> {
        let mut filter = self.filter.write().unwrap();
        
        if filter.contains(key) {
            // Might be duplicate (false positive possible)
            Ok(true)
        } else {
            // Definitely not seen before
            filter.insert(key);
            Ok(false)
        }
    }
}
```

**Benefits**: 10-100× less memory than hash map  
**Drawback**: False positives possible (tune FPR)

---

## Performance Considerations

### Deduplication Cost

| Strategy | CPU Cost | Memory Cost | I/O Cost | Space Savings |
|----------|----------|-------------|----------|---------------|
| **No Dedup** | None | None | Low | 0% |
| **Hash-Based** | Low (hash) | High (map) | Low | 50-90% |
| **Bloom Filter** | Low (hash) | Very Low | Low | 40-80% |
| **Content-Defined Chunking** | High (rolling hash) | Medium | Medium | 70-95% |
| **Delta Encoding** | Very High (diff) | Low | Low | 80-98% |

### When to Deduplicate

#### ✅ **Good Cases**

- **High duplication rate** (>10%)
- **Read-heavy workloads** (amortize cost)
- **Cold data** (archive, backup)
- **Large datasets** (>100GB)
- **Fixed key space** (configurations, schemas)

#### ❌ **Bad Cases**

- **Low duplication rate** (<5%)
- **Write-heavy workloads** (overhead not worth it)
- **Hot data** (latency-sensitive)
- **Small datasets** (<1GB)
- **Random data** (no duplicates to find)

### Optimization Tips

1. **Lazy Deduplication**: Dedupe during compaction, not on write path
2. **Sampling**: Dedupe only large values (>4KB)
3. **Background Processing**: Dedupe asynchronously
4. **Batching**: Dedupe in batches (amortize cost)
5. **Caching**: Cache dedupe results (LRU cache)

---

## Best Practices

### 1. **Choose the Right Layer**

```rust
// ❌ Bad: Deduplicating at multiple layers unnecessarily
let data = app_dedup(raw_data);              // Layer 1
let data = exactly_once_dedup(data);         // Layer 2 (redundant)
let data = lsm_dedup(data);                  // Layer 3 (redundant)

// ✅ Good: Deduplicate at the right layer
let data = app_dedup(raw_data);              // Semantic dedup
storage.write_with_exactly_once(data);       // Idempotency only
// LSM compaction handles storage dedup automatically
```

### 2. **Deterministic Merge Logic**

```rust
// ❌ Bad: Non-deterministic (random, timestamps from system clock)
fn merge(a: Value, b: Value) -> Value {
    if rand::random() {
        a
    } else {
        b
    }
}

// ✅ Good: Deterministic (stable comparison)
fn merge(a: Value, b: Value) -> Value {
    // Use Lamport timestamps + node ID for tie-breaking
    if (a.lamport_ts, a.node_id) > (b.lamport_ts, b.node_id) {
        a
    } else {
        b
    }
}
```

### 3. **Monitor Deduplication Effectiveness**

```rust
pub struct DeduplicationMetrics {
    total_records: AtomicU64,
    duplicates_found: AtomicU64,
    space_saved_bytes: AtomicU64,
    cpu_time_ms: AtomicU64,
}

impl DeduplicationMetrics {
    pub fn dedup_ratio(&self) -> f64 {
        self.duplicates_found.load(Ordering::Relaxed) as f64
            / self.total_records.load(Ordering::Relaxed) as f64
    }
    
    pub fn space_saved_gb(&self) -> f64 {
        self.space_saved_bytes.load(Ordering::Relaxed) as f64 / 1e9
    }
    
    pub fn cost_per_gb(&self) -> Duration {
        let cpu_ms = self.cpu_time_ms.load(Ordering::Relaxed);
        let space_gb = self.space_saved_gb();
        Duration::from_millis((cpu_ms as f64 / space_gb) as u64)
    }
}
```

### 4. **Test Edge Cases**

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_concurrent_deduplication() {
        // Multiple writers, same keys
    }
    
    #[test]
    fn test_out_of_order_writes() {
        // LSN=100, then LSN=50 - keep LSN=100
    }
    
    #[test]
    fn test_tombstone_resurrection() {
        // Write → Delete → Write again
    }
    
    #[test]
    fn test_dedup_cache_eviction() {
        // LRU eviction doesn't cause incorrect dedup
    }
    
    #[test]
    fn test_epoch_rollover() {
        // Epoch changes mid-stream
    }
}
```

---

## Conclusion

Pyralog's **multi-layer deduplication** provides:

✅ **Correctness**: Exactly-once semantics, idempotent writes  
✅ **Efficiency**: 50-95% space savings depending on workload  
✅ **Performance**: Tunable trade-offs per layer  
✅ **Flexibility**: Choose strategies per use case  

**Key Takeaway**: Deduplication is not one-size-fits-all. Choose the right layer and strategy for your workload.

---

## References

- **LSM Compaction**: "The Log-Structured Merge-Tree (LSM-Tree)" - O'Neil et al. (1996)
- **MVCC**: "Multiversion Concurrency Control" - Bernstein & Goodman (1983)
- **Content-Defined Chunking**: "A Low-bandwidth Network File System" - Muthitacharoen et al. (2001)
- **Exactly-Once Semantics**: "Exactly-once Semantics Are Possible" - Kleppmann & Kreps (2021)
- **Bloom Filters**: "Space/Time Trade-offs in Hash Coding with Allowable Errors" - Bloom (1970)

---

**Built with ❤️ for Pyralog**

