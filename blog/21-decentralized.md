# From Cluster to Network: Decentralized Autonomous Databases

**Scale from one datacenter to a global decentralized network**

*Published: November 3, 2025*

---

## The Scaling Hierarchy

Most databases stop at one datacenter:

```
Traditional: Single cluster only
PostgreSQL: One datacenter, strong consistency
MongoDB: Replica sets, one region
Cassandra: Multiple DCs, but no true decentralization

Problem: Geographic distribution = complexity + weak guarantees
```

**Pyralog scales through a hierarchy:**

```
Level 1: Pyralog Cluster (1 datacenter)
  ├─ Strong consistency (Raft)
  ├─ Low latency (<1ms)
  └─ 500M writes/sec

Level 2: Pyralog Network (Multiple clusters)
  ├─ Eventual consistency (Gossip)
  ├─ Global distribution
  └─ Decentralized Autonomous Database
```

---

## Level 1: Pyralog Cluster

**Definition**: Single distributed computing cluster in one datacenter

### Architecture

```
┌────────────────────────────────────────────────┐
│    🔺 PYRALOG CLUSTER (Datacenter US-West)     │
├────────────────────────────────────────────────┤
│                                                │
│  🗿 Pharaoh Network (Obelisk Nodes)           │
│  ├─ Scarab ID generation                      │
│  ├─ Session IDs, epochs                       │
│  └─ Coordination layer                        │
│           ↓ provides IDs                       │
│  🔺 Pyramid Nodes (Storage & Compute)         │
│  ├─ LSM-Tree storage                          │
│  ├─ Raft consensus (per partition)            │
│  ├─ DataFusion queries                        │
│  └─ Batuta execution                          │
│                                                │
│  Properties:                                   │
│  • Strong consistency                          │
│  • ACID transactions                           │
│  • p99 < 1ms latency                          │
│  • 500M writes/sec                            │
│                                                │
└────────────────────────────────────────────────┘
```

### Use Cases

✅ **Perfect for:**
- Single region applications
- Financial systems (need ACID)
- Real-time analytics
- Traditional distributed database needs

### Limitations

⚠️ **Limited by:**
- Geographic latency (cross-continent: 100-300ms)
- Datacenter capacity (network, power)
- Single point of failure (datacenter outage)

---

## Level 2: Pyralog Network

**Definition**: Multiple Pyralog Clusters forming a decentralized autonomous database

### Architecture

```
┌───────────────────────────────────────────────────────────┐
│            🌐 PYRALOG NETWORK (Global)                     │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  🔺 Cluster 1 (US-West)  ←→  🔺 Cluster 2 (US-East)     │
│      ↕ gossip                     ↕ gossip               │
│  🔺 Cluster 3 (EU)       ←→  🔺 Cluster 4 (Asia)        │
│                                                           │
│  Cross-Cluster Communication:                            │
│  ├─ Gossip protocol (epidemic broadcast)                │
│  ├─ Merkle trees (state verification)                   │
│  ├─ CRDTs (conflict-free replicated data)               │
│  └─ Optional: PoW/PoS consensus                         │
│                                                           │
│  Properties:                                              │
│  • Eventual consistency                                  │
│  • Byzantine fault tolerance                             │
│  • Geographic distribution                               │
│  • Decentralized governance                              │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Key Differences

| Feature | Pyralog Cluster | Pyralog Network |
|---------|----------------|-----------------|
| **Geography** | One datacenter | Multiple datacenters |
| **Consistency** | Strong (Raft) | Eventual (Gossip) |
| **Latency** | <1ms | 100-300ms |
| **Consensus** | Raft per partition | Gossip + optional PoW/PoS |
| **Fault Model** | Crash faults | Byzantine faults |
| **Use Case** | Regional DB | Global decentralized DB |

---

## Decentralized Consensus Mechanisms

### Option 1: Gossip Only (Default)

**For**: Trusted participants (all clusters owned by you)

```rust
/// Gossip-based cross-cluster replication
pub struct ClusterGossip {
    /// Local cluster ID
    cluster_id: ClusterId,
    
    /// Known clusters
    peers: HashMap<ClusterId, ClusterInfo>,
    
    /// Gossip interval
    interval: Duration,
}

impl ClusterGossip {
    /// Broadcast updates to peer clusters
    pub async fn gossip_tick(&mut self) {
        // Select random peer clusters
        let targets = self.select_random_peers(3);
        
        // Send state digest
        for peer in targets {
            let digest = self.create_digest();
            self.send_to_peer(peer, digest).await;
        }
    }
}
```

**Properties**:
- ✅ Simple, lightweight
- ✅ Eventually consistent
- ✅ Good for trusted networks
- ⚠️ No Byzantine fault tolerance

---

### Option 2: Proof of Work (PoW)

**For**: Permissionless networks, anti-spam, useful computation

#### Use Cases (NOT Mining!)

**1. Rate Limiting**
```rust
/// Client must solve PoW puzzle to submit request
pub struct RateLimiter {
    difficulty: u32, // Number of leading zeros
}

impl RateLimiter {
    pub fn verify_pow(&self, req: &Request) -> bool {
        let hash = blake3::hash(&req.data);
        count_leading_zeros(hash) >= self.difficulty
    }
}

// Example: 20-bit difficulty = ~1 million hashes (1ms on modern CPU)
// Legitimate users: Fine
// DDoS attackers: CPU cost too high
```

**2. Anti-Spam**
```rust
/// Email-style hashcash for write requests
pub fn submit_with_pow(data: &[u8]) -> Request {
    let mut nonce = 0u64;
    loop {
        let hash = blake3::keyed_hash(&nonce.to_le_bytes(), data);
        if count_leading_zeros(hash) >= DIFFICULTY {
            return Request { data, nonce, hash };
        }
        nonce += 1;
    }
}
```

**3. Priority Queues**
```rust
/// Higher PoW = higher priority
pub fn priority_from_pow(hash: &[u8; 32]) -> Priority {
    Priority(count_leading_zeros(hash))
}

// Users can "pay" CPU for faster processing
```

**4. Time-Lock Puzzles**
```rust
/// Data encrypted with time-lock (future release)
pub fn time_lock_encrypt(data: &[u8], delay: Duration) -> Vec<u8> {
    let difficulty = delay.as_secs() * CPU_HASH_RATE;
    // Encrypt such that decryption requires 'difficulty' hashes
    // Used for scheduled reveals, auctions, etc.
}
```

**Properties**:
- ✅ Permissionless (anyone can participate)
- ✅ Sybil resistant (CPU cost per identity)
- ✅ Useful work (not wasted on mining)
- ⚠️ Energy cost (but much less than Bitcoin)

---

### Option 3: Proof of Stake (PoS)

**For**: Energy-efficient, fast finality, economic security

```rust
/// Stake-based validator selection
pub struct ValidatorSet {
    /// Validators and their stakes
    validators: HashMap<ValidatorId, Stake>,
    
    /// Total staked amount
    total_stake: u128,
}

impl ValidatorSet {
    /// Select validator proportional to stake
    pub fn select_proposer(&self, round: u64) -> ValidatorId {
        let target = (hash(round) % self.total_stake) as u128;
        
        let mut cumulative = 0u128;
        for (id, stake) in &self.validators {
            cumulative += stake.amount;
            if cumulative >= target {
                return *id;
            }
        }
        unreachable!()
    }
    
    /// Slash validator for misbehavior
    pub fn slash(&mut self, validator: ValidatorId, amount: u128) {
        if let Some(stake) = self.validators.get_mut(&validator) {
            stake.amount = stake.amount.saturating_sub(amount);
            stake.slashed += amount;
        }
    }
}
```

**Properties**:
- ✅ Energy efficient (no mining)
- ✅ Fast finality (~3-6 seconds)
- ✅ Economic security (slashing)
- ⚠️ Requires economic value (stake token)

---

## Byzantine Fault Tolerance

### Threat Model

**Crash faults** (Cluster):
- Node crashes
- Network partition
- Disk failure

**Byzantine faults** (Network):
- Malicious nodes
- Arbitrary behavior
- Coordinated attacks

### BFT Consensus

```rust
/// Byzantine-resistant consensus (PBFT-style)
pub struct BFTConsensus {
    /// Replicas (need 3f+1 for f failures)
    replicas: Vec<ReplicaId>,
    
    /// Current view
    view: u64,
    
    /// Quorum size (2f+1)
    quorum: usize,
}

impl BFTConsensus {
    /// Three-phase commit
    pub async fn propose(&mut self, value: Value) -> Result<()> {
        // Phase 1: Pre-prepare
        let preprepare = PrePrepare { view: self.view, value };
        self.broadcast(Message::PrePrepare(preprepare)).await;
        
        // Phase 2: Prepare (wait for 2f responses)
        let prepares = self.collect_prepares(self.quorum).await?;
        
        // Phase 3: Commit (wait for 2f responses)
        let commits = self.collect_commits(self.quorum).await?;
        
        // Committed!
        Ok(())
    }
}
```

**Result**: Tolerates f Byzantine failures with 3f+1 replicas

---

## Real-World Architecture

### Global E-Commerce Platform

```
┌──────────────────────────────────────────────────────┐
│         PYRALOG NETWORK FOR E-COMMERCE                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  🔺 Cluster US-West (Primary)                       │
│  ├─ Strong consistency                              │
│  ├─ Handle: Orders, payments (ACID)                 │
│  └─ Latency: <1ms for West Coast users             │
│                                                      │
│  🔺 Cluster US-East (Replica)                       │
│  ├─ Read replicas                                   │
│  ├─ Handle: Reads for East Coast                    │
│  └─ Eventual consistency: ~100ms lag                │
│                                                      │
│  🔺 Cluster EU (Regional)                           │
│  ├─ Strong consistency (EU data)                    │
│  ├─ GDPR compliance (data locality)                 │
│  └─ Cross-cluster sync: Eventual                    │
│                                                      │
│  🔺 Cluster Asia (Regional)                         │
│  ├─ Strong consistency (Asia data)                  │
│  ├─ Low latency for Asian users                     │
│  └─ Cross-cluster sync: Eventual                    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Strategy**:
- Transactions within cluster (strong consistency)
- Replication across clusters (eventual consistency)
- Conflict resolution via CRDTs
- Users routed to nearest cluster

---

## Migration Path

### Stage 1: Single Cluster

```rust
// Start simple: One cluster
let config = PyralogCluster {
    region: "us-west-1",
    pyramid_nodes: 100,
    obelisk_nodes: 5,
    consistency: Consistency::Strong,
};
```

### Stage 2: Add Read Replicas

```rust
// Add read replicas in other regions
let network = PyralogNetwork::new();
network.add_cluster("us-west-1", ClusterRole::Primary);
network.add_cluster("us-east-1", ClusterRole::ReadReplica);
network.add_cluster("eu-central", ClusterRole::ReadReplica);
```

### Stage 3: Multi-Primary

```rust
// Multiple writable clusters
network.add_cluster("us-west-1", ClusterRole::Primary);
network.add_cluster("eu-central", ClusterRole::Primary);
network.add_cluster("ap-southeast", ClusterRole::Primary);

// Configure conflict resolution
network.set_conflict_resolution(ConflictResolution::LastWriteWins);
```

### Stage 4: Full Decentralization

```rust
// Decentralized autonomous network
network.enable_byzantine_tolerance(true);
network.set_consensus(ConsensusProtocol::PoS);
network.enable_cross_cluster_transactions(true);
```

---

## Performance Comparison

### Cluster (Single DC)

```
Writes: 500M/sec
Reads: 2B/sec
Latency: p99 < 1ms
Consistency: Strong (Raft)
Availability: 99.99% (4 nines)
```

### Network (Multi-DC)

```
Writes: 100M/sec per cluster
Reads: 2B/sec per cluster
Latency: 100-300ms (cross-cluster)
Consistency: Eventual (configurable delay)
Availability: 99.999% (5 nines, geo-redundant)
```

---

## Summary

Pyralog scales from **single cluster** to **global decentralized network**:

### Pyralog Cluster
- ✅ Strong consistency
- ✅ ACID transactions
- ✅ Low latency (<1ms)
- ⚠️ Single datacenter

### Pyralog Network
- ✅ Global distribution
- ✅ Byzantine fault tolerance
- ✅ Decentralized governance
- ⚠️ Eventual consistency

### Consensus Options
1. **Gossip**: Simple, trusted
2. **PoW**: Permissionless, useful work
3. **PoS**: Energy-efficient, economic security

### The Bottom Line

**Start with a cluster. Scale to a network.**

Pyralog's hierarchical architecture lets you start simple (single cluster) and scale globally (decentralized network) without rewriting your application. Strong consistency where you need it, eventual consistency where you can tolerate it.

*From one datacenter to the world.*

---

## Next Steps

- Read [Decentralized Architecture](../DECENTRALIZED.md) for complete details
- See [Nodes Documentation](../NODES.md) for Pyramid/Obelisk architecture
- Check [Shen Ring](12-shen-ring.md) for distributed coordination

---

*Part 21 of the Pyralog Blog Series*

*Previously: [LSM Trees Meet Arrow](20-lsm-arrow.md)*
*Next: [Zero-Knowledge Proofs](22-zk-proofs.md)*

