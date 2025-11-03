# 𓍶 The Shen Ring Architecture

**The eternal circle that binds all distributed system patterns in Pyralog**

---

## Overview

The **Shen Ring** (𓍶) represents the unifying architectural principle in Pyralog: **all distributed coordination patterns can be expressed as variations of circular topology**. Named after the ancient Egyptian symbol for eternity and infinity, the Shen Ring is not a single implementation but a family of ring-based patterns that work together to create a resilient, scalable distributed system.

> *"One ring to rule them all, one ring to find them, one ring to bring them all, and in the distributed system bind them."*

---

## Table of Contents

- [The Philosophy](#the-philosophy)
- [The Five Rings](#the-five-rings)
- [Ring Implementations](#ring-implementations)
  - [1. Ankh Ring - Consistent Hashing](#1-ankh-ring---consistent-hashing)
  - [2. Sundial Circle - Gossip Protocol](#2-sundial-circle---gossip-protocol)
  - [3. Cartouche Ring - Token Coordination](#3-cartouche-ring---token-coordination)
  - [4. Ouroboros Circle - Chain Replication](#4-ouroboros-circle---chain-replication)
  - [5. Shen Ring - The Unifying Log](#5-shen-ring---the-unifying-log)
- [Architectural Integration](#architectural-integration)
- [Implementation Patterns](#implementation-patterns)
- [Performance Characteristics](#performance-characteristics)
- [Fault Tolerance](#fault-tolerance)
- [References](#references)

---

## The Philosophy

### Why Rings?

Rings are fundamental to distributed systems because they provide:

1. **No Single Point of Failure** - Every node is equal in the topology
2. **Predictable Routing** - O(log N) or O(1) navigation with proper indexing
3. **Natural Load Distribution** - Consistent hashing ensures even distribution
4. **Self-Healing Properties** - Ring structure enables automatic rebalancing
5. **Elegance** - Simple, beautiful abstractions for complex problems

### The Eternal Circle

In ancient Egypt, the Shen ring (𓍶) symbolized:
- **Eternity** - No beginning, no end
- **Protection** - Encircling and safeguarding
- **Infinity** - Continuous, unbroken cycle
- **Unity** - All parts connected as one

These same properties define Pyralog's ring architecture.

---

## The Five Rings

| Ring | Icon | Purpose | Pattern | Scope |
|------|------|---------|---------|-------|
| **Ankh Ring** | ☥ | Partition Assignment | Consistent Hashing | Cluster-wide |
| **Sundial Circle** | ⭕ | State Synchronization | Gossip Protocol | Peer-to-peer |
| **Cartouche Ring** | 𓍹𓍺 | Mutual Exclusion | Token Passing | Cross-partition |
| **Ouroboros Circle** | 🐍 | Data Replication | Chain Replication | Per-partition |
| **Shen Ring** | 𓍶 | Unified Interface | Append-Only Log | Application |

Each ring serves a specific purpose while sharing common principles:
- **Circular topology** for resilience
- **Deterministic routing** for predictability  
- **Local state** with global view
- **Self-stabilization** after failures

---

## Ring Implementations

### 1. Ankh Ring - Consistent Hashing

**Purpose**: Distribute partitions across Pharaoh nodes using consistent hashing

#### Overview

The Ankh (☥), symbol of life, represents the living, breathing distribution of data across the cluster. As nodes join and leave, the ring rebalances automatically, maintaining system vitality.

#### Algorithm

```rust
/// Ankh Ring: Consistent hash ring for partition assignment
pub struct AnkhRing {
    /// Virtual nodes on the ring (160 per physical node)
    vnodes: BTreeMap<Hash, NodeId>,
    /// Physical node metadata
    nodes: HashMap<NodeId, NodeInfo>,
    /// Hash function (XXH3 for speed)
    hasher: xxhash_rust::xxh3::Xxh3,
}

impl AnkhRing {
    /// Map a Scarab ID to a set of replica nodes
    pub fn locate(&self, key: ScarabId) -> Vec<NodeId> {
        let hash = self.hash_key(key);
        
        // Find the first vnode >= hash
        let primary = self.vnodes
            .range(hash..)
            .next()
            .or_else(|| self.vnodes.iter().next())
            .map(|(_, node)| *node)
            .expect("ring cannot be empty");
        
        // Walk clockwise to find N distinct physical nodes
        self.walk_ring(hash, REPLICATION_FACTOR)
    }
    
    /// Add a new node to the ring
    pub fn add_node(&mut self, node: NodeId, vnodes: u32) {
        for i in 0..vnodes {
            let vnode_key = format!("{}:{}", node, i);
            let hash = self.hash(&vnode_key);
            self.vnodes.insert(hash, node);
        }
    }
    
    /// Remove a node and return affected key ranges
    pub fn remove_node(&mut self, node: NodeId) -> Vec<(Hash, Hash)> {
        let affected_ranges = self.vnodes
            .iter()
            .filter(|(_, n)| **n == node)
            .map(|(hash, _)| *hash)
            .collect::<Vec<_>>();
        
        for hash in &affected_ranges {
            self.vnodes.remove(hash);
        }
        
        self.compute_rebalance_ranges(&affected_ranges)
    }
}
```

#### Key Properties

- **Virtual Nodes**: 160 vnodes per physical node for even distribution
- **Hash Function**: XXH3 (fast, high-quality distribution)
- **Replication**: RF=3 by default, walking clockwise on ring
- **Rebalancing**: Only affected ranges move when nodes join/leave

#### Example

```rust
let mut ring = AnkhRing::new();

// Add three Pharaoh nodes
ring.add_node(NodeId(1), 160);
ring.add_node(NodeId(2), 160);
ring.add_node(NodeId(3), 160);

// Locate partition for a Scarab ID
let key = ScarabId::new();
let replicas = ring.locate(key);
// => [NodeId(2), NodeId(3), NodeId(1)]  // Primary + 2 replicas

// Node 2 fails - automatic rebalancing
ring.remove_node(NodeId(2));
let new_replicas = ring.locate(key);
// => [NodeId(3), NodeId(1), NodeId(4)]  // Seamlessly shifts
```

---

### 2. Sundial Circle - Gossip Protocol

**Purpose**: Peer-to-peer state synchronization and failure detection

#### Overview

Like the sun moving across an ancient sundial (⭕), information spreads gradually but inevitably across all nodes. Each node gossips with neighbors, ensuring eventual consistency without centralized coordination.

#### Protocol

```rust
/// Sundial Circle: Epidemic gossip for cluster membership
pub struct SundialCircle {
    /// Local node information
    local: NodeState,
    /// Known peers and their state
    peers: HashMap<NodeId, PeerState>,
    /// Gossip fanout (how many peers to gossip with)
    fanout: usize,
    /// Gossip interval
    interval: Duration,
}

#[derive(Clone, Debug)]
pub struct PeerState {
    /// Node metadata
    node: NodeInfo,
    /// Heartbeat counter (monotonically increasing)
    heartbeat: u64,
    /// Last time we heard from this node
    last_seen: Instant,
    /// Suspected of failure?
    suspected: bool,
}

impl SundialCircle {
    /// Gossip tick - runs periodically
    pub async fn gossip_tick(&mut self) {
        // Increment local heartbeat
        self.local.heartbeat += 1;
        
        // Select random peers (prefer recently active)
        let targets = self.select_gossip_targets();
        
        // Send digest to each target
        for peer in targets {
            let digest = self.create_digest();
            let response = self.send_gossip(peer, digest).await;
            
            if let Ok(remote_digest) = response {
                self.merge_state(remote_digest);
            }
        }
        
        // Check for suspected failures
        self.detect_failures();
    }
    
    /// Merge remote state with local state
    fn merge_state(&mut self, remote: Digest) {
        for (node_id, remote_state) in remote.peers {
            match self.peers.get_mut(&node_id) {
                Some(local_state) => {
                    // Keep the state with higher heartbeat
                    if remote_state.heartbeat > local_state.heartbeat {
                        *local_state = remote_state;
                    }
                }
                None => {
                    // New node discovered
                    self.peers.insert(node_id, remote_state);
                }
            }
        }
    }
    
    /// Failure detection based on timeout
    fn detect_failures(&mut self) {
        let now = Instant::now();
        let timeout = Duration::from_secs(10);
        
        for (node_id, state) in &mut self.peers {
            if now.duration_since(state.last_seen) > timeout {
                if !state.suspected {
                    state.suspected = true;
                    self.trigger_suspicion(*node_id);
                }
            }
        }
    }
}
```

#### Key Properties

- **Fanout**: 3-5 peers per gossip round
- **Interval**: 100-500ms between rounds
- **Convergence**: O(log N) rounds to reach all nodes
- **Overhead**: O(N) messages per round across cluster
- **Failure Detection**: Phi-accrual failure detector (adaptive timeouts)

#### Gossip Patterns

```rust
// Push gossip: Send full state
async fn push_gossip(&self, peer: NodeId) {
    let state = self.local.clone();
    self.send(peer, GossipMessage::Push(state)).await;
}

// Pull gossip: Request remote state
async fn pull_gossip(&self, peer: NodeId) {
    let digest = self.create_digest();
    let response = self.send(peer, GossipMessage::Pull(digest)).await;
    self.merge_state(response);
}

// Push-Pull gossip: Exchange state (most efficient)
async fn push_pull_gossip(&mut self, peer: NodeId) {
    let digest = self.create_digest();
    let response = self.send(peer, GossipMessage::PushPull(digest)).await;
    
    // Send differences back
    let delta = self.compute_delta(&response);
    self.send(peer, GossipMessage::Delta(delta)).await;
    
    self.merge_state(response);
}
```

---

### 3. Cartouche Ring - Token Coordination

**Purpose**: Distributed mutual exclusion using token passing

#### Overview

A cartouche (𓍹𓍺) encloses a pharaoh's name, protecting it from chaos. The Cartouche Ring protects critical operations by passing a token that grants exclusive access.

#### Algorithm

```rust
/// Cartouche Ring: Token-based mutual exclusion
pub struct CartoucheRing {
    /// Ordered list of nodes in the ring
    nodes: Vec<NodeId>,
    /// Current token holder (None if token is lost)
    token_holder: Option<NodeId>,
    /// Token sequence number (for detecting duplicates)
    token_seq: u64,
    /// Pending requests queue
    requests: VecDeque<TokenRequest>,
}

#[derive(Clone, Debug)]
pub struct Token {
    /// Unique sequence number
    seq: u64,
    /// Current holder
    holder: NodeId,
    /// Request queue (carried with token)
    queue: Vec<TokenRequest>,
    /// Timestamp for timeout detection
    issued_at: Instant,
}

impl CartoucheRing {
    /// Request the token for a critical operation
    pub async fn request_token(&mut self) -> TokenGuard {
        let request = TokenRequest {
            node: self.local_node_id(),
            timestamp: Instant::now(),
        };
        
        // Add to local queue
        self.requests.push_back(request.clone());
        
        // If we have the token, use it immediately
        if self.has_token() {
            return self.acquire_token();
        }
        
        // Otherwise, propagate request around ring
        self.send_request(request).await;
        
        // Wait for token arrival
        self.wait_for_token().await
    }
    
    /// Release the token and pass to next requester
    pub async fn release_token(&mut self, guard: TokenGuard) {
        drop(guard); // Ensure exclusive access is dropped
        
        let next = self.next_requester();
        
        if let Some(next_node) = next {
            self.pass_token(next_node).await;
        } else {
            // No pending requests - keep token
            self.token_holder = Some(self.local_node_id());
        }
    }
    
    /// Token passing logic
    async fn pass_token(&mut self, target: NodeId) {
        let token = Token {
            seq: self.token_seq,
            holder: target,
            queue: self.requests.drain(..).collect(),
            issued_at: Instant::now(),
        };
        
        self.send(target, RingMessage::Token(token)).await;
        self.token_holder = Some(target);
    }
    
    /// Regenerate lost token (requires consensus)
    async fn regenerate_token(&mut self) {
        // Use Sundial Circle to elect token regenerator
        let elected = self.sundial.elect_leader().await;
        
        if elected == self.local_node_id() {
            warn!("Regenerating lost token");
            let new_token = Token {
                seq: self.token_seq + 1,
                holder: self.local_node_id(),
                queue: Vec::new(),
                issued_at: Instant::now(),
            };
            
            self.token_seq += 1;
            self.inject_token(new_token);
        }
    }
}

/// RAII guard for token-based critical section
pub struct TokenGuard {
    ring: Arc<Mutex<CartoucheRing>>,
}

impl Drop for TokenGuard {
    fn drop(&mut self) {
        // Automatically release token when guard is dropped
        let mut ring = self.ring.lock().unwrap();
        ring.release_token_internal();
    }
}
```

#### Key Properties

- **Fairness**: FIFO order based on timestamps
- **Safety**: Only one token holder at a time
- **Liveness**: Token keeps moving if requests exist
- **Fault Tolerance**: Token regeneration if lost
- **Timeout**: Token reclaimed after 10s if holder crashes

#### Use Cases

```rust
// Example: Distributed schema migration
async fn update_schema(&self, migration: Migration) {
    // Acquire token for exclusive cluster-wide access
    let _guard = self.cartouche.request_token().await;
    
    // Perform schema update (no other node can modify schema)
    self.apply_migration(migration).await;
    
    // Token automatically released when guard is dropped
}
```

---

### 4. Ouroboros Circle - Chain Replication

**Purpose**: Strong consistency through ordered chain replication

#### Overview

The Ouroboros (🐍), a serpent eating its own tail, represents the eternal cycle of data flowing through replicas and returning to the head. This pattern provides linearizable reads and writes.

#### Protocol

```rust
/// Ouroboros Circle: Chain replication with wraparound
pub struct OuroborosCircle {
    /// Ordered chain of replicas
    chain: Vec<NodeId>,
    /// Am I the head?
    is_head: bool,
    /// Am I the tail?
    is_tail: bool,
    /// Pending writes (head only)
    pending: HashMap<Offset, WriteRequest>,
    /// Acknowledged writes (tail only)
    committed: BTreeSet<Offset>,
}

impl OuroborosCircle {
    /// Write operation (only accepted at head)
    pub async fn write(&mut self, data: Bytes) -> Result<Offset> {
        if !self.is_head {
            return Err(Error::NotHead);
        }
        
        // Assign offset using Obelisk Sequencer
        let offset = self.obelisk.next().await?;
        
        // Create write request
        let request = WriteRequest {
            offset,
            data: data.clone(),
            timestamp: Instant::now(),
        };
        
        self.pending.insert(offset, request.clone());
        
        // Forward to next replica in chain
        self.forward_write(request).await?;
        
        // Wait for acknowledgment from tail
        self.wait_for_commit(offset).await
    }
    
    /// Forward write to next replica
    async fn forward_write(&self, request: WriteRequest) -> Result<()> {
        let next = self.next_replica();
        
        // Write to local storage first
        self.storage.append(request.offset, &request.data).await?;
        
        // Then forward to next in chain
        if let Some(next_node) = next {
            self.send(next_node, ChainMessage::Write(request)).await?;
        } else {
            // We're the tail - send acknowledgment back to head
            self.acknowledge_write(request.offset).await?;
        }
        
        Ok(())
    }
    
    /// Read operation (only served by tail)
    pub async fn read(&self, offset: Offset) -> Result<Bytes> {
        if !self.is_tail {
            return Err(Error::NotTail);
        }
        
        // Tail serves reads - guaranteed to be committed
        self.storage.read(offset).await
    }
    
    /// Handle node failure - reconfigure chain
    pub async fn handle_failure(&mut self, failed: NodeId) {
        if self.is_predecessor_of(failed) {
            // Skip failed node, forward to its successor
            self.chain.retain(|n| *n != failed);
            
            // Resend any pending writes
            for (offset, request) in &self.pending {
                self.forward_write(request.clone()).await.ok();
            }
        }
        
        if failed == self.head() {
            // Head failed - promote first backup to head
            self.promote_to_head().await;
        }
        
        if failed == self.tail() {
            // Tail failed - promote predecessor to tail
            self.promote_to_tail().await;
        }
    }
}
```

#### Chain Topology

```
┌─────────────────────────────────────────┐
│     Ouroboros Chain (RF=3)              │
│                                         │
│   Client                                │
│     │                                   │
│     │ write()                           │
│     ▼                                   │
│   ┌───┐      ┌───┐      ┌───┐         │
│   │ H │─────►│ M │─────►│ T │         │
│   └───┘      └───┘      └───┘         │
│   Head       Middle      Tail          │
│     │                      │           │
│     │◄─────── ack ─────────┘           │
│     │                                   │
│     ▼                                   │
│   Client                                │
│     │                                   │
│     │ read()                            │
│     └──────────────────►│ T │           │
│                         └───┘           │
│                         Tail            │
└─────────────────────────────────────────┘
```

#### Key Properties

- **Linearizability**: Writes ordered at head, reads at tail
- **Low Latency**: Single round-trip to head + chain propagation
- **Strong Durability**: RF replicas before acknowledgment
- **Fast Reads**: Tail serves committed data only
- **Recovery**: Chain reconfiguration on failure

---

### 5. Shen Ring - The Unifying Log

**Purpose**: Application-level abstraction unifying all ring patterns

#### Overview

The Shen Ring (𓍶) is the outermost circle - the eternal interface that applications interact with. It represents the unified append-only log that leverages all other rings internally.

#### Interface

```rust
/// Shen Ring: The One Ring that unifies all distributed patterns
pub struct ShenRing {
    /// Partition assignment (Ankh Ring)
    ankh: Arc<AnkhRing>,
    /// Cluster membership (Sundial Circle)
    sundial: Arc<SundialCircle>,
    /// Global coordination (Cartouche Ring)
    cartouche: Arc<CartoucheRing>,
    /// Replication chain (Ouroboros Circle)
    ouroboros: Arc<OuroborosCircle>,
    /// ID generation (Scarab IDs via Obelisk)
    obelisk: Arc<ObeliskSequencer>,
}

impl ShenRing {
    /// Append a record to the distributed log
    pub async fn append(&self, data: Bytes) -> Result<ScarabId> {
        // 1. Generate globally unique ID (Obelisk)
        let id = self.obelisk.generate_scarab_id().await?;
        
        // 2. Determine partition using consistent hashing (Ankh)
        let partition = self.ankh.locate(id);
        
        // 3. Write to replication chain (Ouroboros)
        let offset = self.ouroboros.write(partition, data).await?;
        
        // 4. Return combined ID
        Ok(ScarabId::new(partition, offset))
    }
    
    /// Read a record from the log
    pub async fn read(&self, id: ScarabId) -> Result<Bytes> {
        // 1. Locate partition (Ankh)
        let replicas = self.ankh.locate(id);
        
        // 2. Read from tail of replication chain (Ouroboros)
        let tail = replicas.last().unwrap();
        self.ouroboros.read(tail, id.offset()).await
    }
    
    /// Execute a distributed transaction across partitions
    pub async fn transaction<F>(&self, f: F) -> Result<()>
    where
        F: FnOnce(&mut Transaction) -> Result<()>,
    {
        // Acquire token for serializable isolation (Cartouche)
        let _guard = self.cartouche.request_token().await;
        
        // Execute transaction with exclusive access
        let mut txn = Transaction::new(self);
        f(&mut txn)?;
        txn.commit().await
    }
    
    /// Get cluster status and health
    pub fn cluster_status(&self) -> ClusterStatus {
        // Use gossip state for membership view (Sundial)
        let peers = self.sundial.get_peers();
        
        ClusterStatus {
            nodes: peers.len(),
            healthy: peers.iter().filter(|p| !p.suspected).count(),
            partitions: self.ankh.partition_count(),
            replication_factor: self.ouroboros.replica_count(),
        }
    }
}
```

#### The Unified Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                       │
│                                                              │
│                  ┌──────────────────┐                        │
│                  │   Batuta Query   │                        │
│                  │    Language 🎼   │                        │
│                  └────────┬─────────┘                        │
│                           │                                  │
└───────────────────────────┼──────────────────────────────────┘
                            │
┌───────────────────────────┼──────────────────────────────────┐
│                           │                                  │
│                  ┌────────▼─────────┐                        │
│                  │   Shen Ring 𓍶   │  ◄── The One Ring     │
│                  │  (Unified Log)   │                        │
│                  └────────┬─────────┘                        │
│                           │                                  │
│            ┌──────────────┼──────────────┐                   │
│            │              │              │                   │
│   ┌────────▼───────┐ ┌───▼────┐ ┌───────▼────────┐          │
│   │  Ankh Ring ☥   │ │Sundial │ │ Cartouche 𓍹𓍺  │          │
│   │  (Partitions)  │ │Circle⭕│ │   (Token)      │          │
│   └────────┬───────┘ └───┬────┘ └───────┬────────┘          │
│            │             │              │                   │
│            └─────────────┼──────────────┘                   │
│                          │                                  │
│                 ┌────────▼─────────┐                         │
│                 │ Ouroboros Loop 🐍 │                         │
│                 │  (Replication)   │                         │
│                 └────────┬─────────┘                         │
│                          │                                  │
└──────────────────────────┼──────────────────────────────────┘
                           │
┌──────────────────────────┼──────────────────────────────────┐
│                 STORAGE LAYER                                │
│                          │                                  │
│            ┌─────────────▼──────────────┐                    │
│            │  Obelisk Sequencer 🗿      │                    │
│            │  (Scarab IDs 🪲)           │                    │
│            └────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Architectural Integration

### How The Rings Work Together

1. **Client Writes**:
   ```
   Client → Shen Ring → Obelisk (ID) → Ankh (partition) 
         → Ouroboros (replicate) → Scarab ID returned
   ```

2. **Client Reads**:
   ```
   Client → Shen Ring → Ankh (locate) → Ouroboros (tail read)
         → Data returned
   ```

3. **Cluster Changes**:
   ```
   Node Join → Sundial (gossip) → Ankh (rebalance)
            → Ouroboros (reconfigure chains)
   ```

4. **Critical Operations**:
   ```
   Admin → Shen Ring → Cartouche (acquire token)
        → Operation → Cartouche (release token)
   ```

### Ring Interaction Matrix

| Operation | Ankh | Sundial | Cartouche | Ouroboros | Shen |
|-----------|------|---------|-----------|-----------|------|
| **Append** | ✓ Locate | - | - | ✓ Write | ✓ Coordinate |
| **Read** | ✓ Locate | - | - | ✓ Read | ✓ Coordinate |
| **Node Join** | ✓ Rebalance | ✓ Discover | - | ✓ Reconfigure | - |
| **Node Fail** | ✓ Rebalance | ✓ Detect | ✓ Regenerate | ✓ Reconfigure | - |
| **Schema Change** | - | - | ✓ Serialize | - | ✓ Coordinate |
| **Transaction** | ✓ Locate | - | ✓ Isolate | ✓ Write | ✓ Coordinate |

---

## Implementation Patterns

### Ring State Management

```rust
/// Common ring state pattern
pub trait RingState {
    type Node;
    type Config;
    
    /// Get ordered list of nodes
    fn nodes(&self) -> &[Self::Node];
    
    /// Find next node clockwise
    fn next(&self, current: &Self::Node) -> Option<&Self::Node>;
    
    /// Find previous node counter-clockwise
    fn prev(&self, current: &Self::Node) -> Option<&Self::Node>;
    
    /// Handle ring reconfiguration
    fn reconfigure(&mut self, config: Self::Config);
}
```

### Ring Communication

```rust
/// Generic ring message passing
pub trait RingMessaging {
    type Message;
    
    /// Send to next node in ring
    async fn forward(&self, msg: Self::Message) -> Result<()>;
    
    /// Broadcast to all nodes
    async fn broadcast(&self, msg: Self::Message) -> Result<()>;
    
    /// Multi-cast to subset of nodes
    async fn multicast(&self, nodes: &[NodeId], msg: Self::Message) -> Result<()>;
}
```

### Ring Fault Tolerance

```rust
/// Fault detection and recovery
pub trait RingFaultTolerance {
    /// Detect failed node
    fn detect_failure(&mut self, node: NodeId);
    
    /// Remove failed node and reconfigure
    fn handle_failure(&mut self, node: NodeId) -> ReconfigurationPlan;
    
    /// Add new node and rebalance
    fn handle_join(&mut self, node: NodeId) -> ReconfigurationPlan;
    
    /// Stabilize after topology change
    async fn stabilize(&mut self) -> Result<()>;
}
```

---

## Performance Characteristics

### Time Complexity

| Operation | Ankh | Sundial | Cartouche | Ouroboros | Shen |
|-----------|------|---------|-----------|-----------|------|
| **Locate** | O(log V) | - | - | O(1) | O(log V) |
| **Route** | O(1) | O(log N) | O(N) | O(RF) | O(log V + RF) |
| **Join** | O(V/N) | O(log N) | O(1) | O(1) | O(V/N + log N) |
| **Leave** | O(V/N) | O(log N) | O(1) | O(1) | O(V/N + log N) |
| **Gossip** | - | O(F) | - | - | - |

Where:
- **V** = Virtual nodes (typically 160 per physical node)
- **N** = Number of physical nodes
- **RF** = Replication factor (typically 3)
- **F** = Gossip fanout (typically 3-5)

### Space Complexity

| Ring | Per-Node Memory | Cluster-Wide |
|------|----------------|--------------|
| **Ankh** | O(V) | O(N × V) |
| **Sundial** | O(N) | O(N²) |
| **Cartouche** | O(R) | O(R) |
| **Ouroboros** | O(P) | O(P × RF) |
| **Shen** | O(V + N + P) | O(N × (V + N + P)) |

Where:
- **R** = Pending requests (bounded by queue size)
- **P** = Partitions per node

### Latency

| Operation | Latency | Factors |
|-----------|---------|---------|
| **Ankh Lookup** | < 1µs | Hash computation + B-tree lookup |
| **Sundial Gossip** | 100-500ms | Gossip interval + fanout |
| **Cartouche Acquire** | 10-100ms | Token round-trip through ring |
| **Ouroboros Write** | 1-5ms | Chain length + network RTT |
| **Shen Append** | 2-10ms | Ankh + Ouroboros combined |

---

## Fault Tolerance

### Failure Modes

| Failure Type | Detection | Recovery | Impact |
|--------------|-----------|----------|--------|
| **Node Crash** | Sundial (10s) | Ankh rebalance + Ouroboros reconfigure | Temporary unavailability |
| **Network Partition** | Sundial (30s) | Quorum-based operation | Split-brain risk |
| **Slow Node** | Ouroboros timeout | Chain bypass | Degraded latency |
| **Lost Token** | Cartouche timeout | Token regeneration | Brief lock unavailability |
| **Cascading Failures** | Sundial mass detection | Graceful degradation | Reduced capacity |

### Recovery Strategies

#### 1. Ankh Ring Recovery

```rust
/// Rebuild consistent hash ring from Sundial membership
async fn recover_ankh_ring(&mut self) {
    let alive_nodes = self.sundial.get_healthy_nodes();
    
    // Rebuild ring with only healthy nodes
    let mut new_ring = AnkhRing::new();
    for node in alive_nodes {
        new_ring.add_node(node, VNODES_PER_NODE);
    }
    
    // Compute data migration plan
    let migrations = self.compute_migrations(&self.ankh, &new_ring);
    
    // Swap in new ring
    self.ankh = Arc::new(new_ring);
    
    // Execute migrations in background
    self.execute_migrations(migrations).await;
}
```

#### 2. Ouroboros Recovery

```rust
/// Reconfigure replication chain after node failure
async fn recover_ouroboros_chain(&mut self, failed: NodeId) {
    // Remove failed node from chain
    self.ouroboros.remove_node(failed);
    
    // Find replacement node from Ankh ring
    let replacement = self.ankh.next_available_node(failed);
    
    // Insert into chain
    self.ouroboros.insert_node(replacement);
    
    // Synchronize data to new replica
    self.sync_replica(replacement).await;
}
```

#### 3. Cartouche Recovery

```rust
/// Regenerate lost token using Sundial consensus
async fn recover_cartouche_token(&mut self) {
    // Use Sundial to elect token generator
    let elected = self.sundial.elect_coordinator().await;
    
    if elected == self.local_id {
        // Generate new token with incremented sequence
        let token = Token {
            seq: self.cartouche.last_seq + 1,
            holder: self.local_id,
            queue: Vec::new(),
            issued_at: Instant::now(),
        };
        
        // Inject token into ring
        self.cartouche.inject_token(token);
    }
}
```

---

## References

### Academic Papers

1. **Consistent Hashing**: Karger et al. "Consistent Hashing and Random Trees" (1997)
2. **Gossip Protocols**: Demers et al. "Epidemic Algorithms for Replicated Database Maintenance" (1987)
3. **Token Ring**: Tanenbaum "Distributed Operating Systems" (1995)
4. **Chain Replication**: van Renesse & Schneider "Chain Replication for Supporting High Throughput and Availability" (2004)
5. **Distributed Logs**: Kreps et al. "Kafka: A Distributed Messaging System for Log Processing" (2011)

### Systems

- **Cassandra**: Ankh Ring (consistent hashing) + Sundial (gossip)
- **Riak**: Ankh Ring + vector clocks
- **CRAQ**: Ouroboros (chain replication with apportioned queries)
- **Kafka**: Shen Ring (distributed log abstraction)
- **DynamoDB**: Ankh Ring + Sundial + vector clocks

### Further Reading

- [Pyralog Obelisk Sequencer](OBELISK.md) - Crash-safe ID generation
- [Pyralog Pharaoh Network](PHARAOH.md) - Distributed coordination
- [Pyralog Scarab IDs](SCARAB.md) - Globally unique identifiers
- [Pyralog Batuta Language](BATUTA.md) - Query and programming language

---

## License

MIT-0 (MIT No Attribution)

Copyright 2025 Pyralog Contributors

---

**The Shen Ring** - *One ring to bind them all in distributed harmony* 𓍶

