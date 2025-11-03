# Pyralog Documentation Index

**Complete guide to Pyralog: A platform for secure, parallel, distributed, and decentralized computing.**

Pyralog unifies cryptographic verification, actor-based concurrency, functional programming, multi-model databases, and decentralized consensus into a single coherent system.

---

## 📚 Quick Navigation

### 🚀 Getting Started
| Document | Description |
|----------|-------------|
| [README](README.md) | Project overview and features |
| [DOCUMENTATION_STATISTICS](DOCUMENTATION_STATISTICS.md) 📊 | Complete stats (144 files, 94K lines, 328K words, ~27h reading time) |
| [QUICK_START](QUICK_START.md) | Get running in 5 minutes |
| [FAQ](FAQ.md) | Frequently asked questions |
| [EXAMPLES](EXAMPLES.md) | Practical code examples |

### 📊 Visual Architecture
| Diagram | Focus |
|---------|-------|
| [System Architecture](diagrams/system-architecture.mmd) | Complete platform overview |
| [Shen Ring](diagrams/shen-ring.mmd) | Five rings architecture (Ankh, Sundial, Cartouche, Ouroboros, Shen) |
| [Data Flow](diagrams/data-flow.mmd) | Write/read paths and background processes |
| [Component Relationships](diagrams/component-relationships.mmd) | How all pieces fit together |
| [Actor Topology](diagrams/actor-topology.mmd) | Supervision trees and location transparency |
| [Consensus](diagrams/consensus.mmd) | Raft protocol flow |
| [LSM Storage](diagrams/lsm-storage.mmd) | Log-structured merge tree |
| [PPHM Build](diagrams/pphm-build.mmd) | Perfect hash map pipeline |
| [Deduplication Layers](diagrams/deduplication-layers.mmd) | Multi-layer strategy |
| [Exactly-Once Semantics](diagrams/exactly-once.mmd) | Session-based idempotent writes |

### 🏗️ Core Architecture
| Document | Focus |
|----------|-------|
| [CORE_CONCEPTS](CORE_CONCEPTS.md) | Fundamentals (LogId, Partitions, Records, Offsets, Epochs) |
| [NODES](NODES.md) ⭐ | Two-tier architecture (Obelisk nodes, Pyramid nodes, Pharaoh Network) |
| [ARCHITECTURE](ARCHITECTURE.md) | Deep dive into system internals |
| [DESIGN](DESIGN.md) | Design decisions and rationale |
| [SHEN_RING](SHEN_RING.md) ⭐ | Five Rings (Ankh, Sundial, Cartouche, Ouroboros, Shen) |
| [DATA_PATH](DATA_PATH.md) | Write and read paths with diagrams |
| [EPOCHS](EPOCHS.md) | Epochs and sequencers |
| [CAP_THEOREM](CAP_THEOREM.md) | Consistency, availability, tradeoffs |
| [DYNAMIC_PARTITIONS](DYNAMIC_PARTITIONS.md) | Dynamic partition splitting/merging |
| [CLIENT_PARTITIONING_PATTERNS](CLIENT_PARTITIONING_PATTERNS.md) | Client-side partitioning strategies |

### 🎯 Novel Innovations
| Document | Innovation |
|----------|-----------|
| [PAPER](PAPER.md) ⭐ | Academic research paper (40+ pages, 23 references) |
| [PPHM](PPHM.md) ⭐ | Partitioned Perfect Hash Maps (O(1) lookups, zero collisions) |
| [DEDUPLICATION](DEDUPLICATION.md) ⭐ | Multi-layer deduplication strategy |

### 🗄️ Storage & Data
| Document | Focus |
|----------|-------|
| [STORAGE](STORAGE.md) ⭐ | LSM-Tree storage engine + hybrid architecture |
| [MEMORY_ONLY_MODE](MEMORY_ONLY_MODE.md) ⭐ | Ultra-fast ephemeral storage (10-100× faster) |
| [ARROW](ARROW.md) ⭐ | Apache Arrow in Rust (columnar, zero-copy) |
| [DATA_FORMATS](DATA_FORMATS.md) ⭐ | Parquet, Safetensors, Zarr, DLPack |
| [TENSOR_DATABASE](TENSOR_DATABASE.md) ⭐ | Multi-dimensional arrays, ML/AI, vectors, embeddings |
| [MULTI_MODEL_DATABASE](MULTI_MODEL_DATABASE.md) ⭐ | Relational, document, graph, RDF, key-value (Category Theory) |
| [IMMUTABLE_KNOWLEDGE_DB](IMMUTABLE_KNOWLEDGE_DB.md) ⭐ | Temporal knowledge systems (EAVT, time-travel queries) |

### 🔒 Security & Cryptography
| Document | Focus |
|----------|-------|
| [CRYPTOGRAPHIC_VERIFICATION](CRYPTOGRAPHIC_VERIFICATION.md) ⭐ | Merkle trees, zero-trust, BLAKE3, notarization, HSM |
| [WIREGUARD_PROTOCOL](WIREGUARD_PROTOCOL.md) ⭐ | Quantum-resistant protocol (WireGuard + Rosenpass, DPI evasion) |

### 🌐 Decentralized Systems
| Document | Focus |
|----------|-------|
| [DECENTRALIZED](DECENTRALIZED.md) ⭐ | Cluster vs Network hierarchy, PoW, PoS, zk-SNARKs, zk-STARKs |
| [DADBS](DADBS.md) ⭐ | Decentralized Autonomous Database Systems (5 consensus mechanisms) |

### 🎭 Concurrency & Distribution
| Document | Focus |
|----------|-------|
| [ACTOR_MODEL](ACTOR_MODEL.md) ⭐ | Location-transparent actors, supervision trees, formal semantics |
| [FUNCTIONAL_RELATIONAL_ALGEBRA](FUNCTIONAL_RELATIONAL_ALGEBRA.md) ⭐ | Pure functional query system (monads, functors) |

### 💬 Query & Programming Languages
| Language | Type | Focus |
|----------|------|-------|
| [BATUTA](BATUTA.md) ⭐ | Full Programming Language | Category Theory, Functional Relational Algebra, actor-first |
| [PRQL](PRQL.md) ⭐ | Query Language | Pragmatic, readable SQL alternative (pipelines, composable) |
| [GraphQL](GRAPHQL.md) ⭐ | API Query Language | Client-driven, type-safe, real-time subscriptions |
| [JSON-RPC/WebSocket](JSONRPC_WEBSOCKET.md) ⭐ | RPC Protocol | Low-latency (<5ms), bidirectional, binary support |

**Quick Decision Guide**:
- **Full applications with business logic?** → **Batuta** (Category Theory, formal guarantees)
- **Readable relational queries?** → **PRQL** (pragmatic, compiles to SQL)
- **Flexible API layer?** → **GraphQL** (client-driven, nested queries)
- **Low-latency RPC?** → **JSON-RPC/WebSocket** (<5ms, real-time)

**Theoretical Rigor**: SQL (none) < PRQL (pragmatic) < **Batuta (Category Theory)**

### 📈 Advanced Features
| Document | Focus |
|----------|-------|
| [ADVANCED_FEATURES](ADVANCED_FEATURES.md) | Transactions (Percolator), exactly-once, stream processing, time-travel |

### 🛠️ Operations
| Document | Focus |
|----------|-------|
| [OPERATIONS](OPERATIONS.md) | Deployment, configuration, maintenance |
| [PERFORMANCE](PERFORMANCE.md) | Performance tuning and optimization |
| [COMPARISON](COMPARISON.md) | vs Kafka, Redpanda, LogDevice, Pulsar |
| [TIKV_COMPARISON](TIKV_COMPARISON.md) | Detailed comparison with TiKV |

### 🧑‍💻 Development
| Document | Focus |
|----------|-------|
| [CONTRIBUTING](CONTRIBUTING.md) | How to contribute to Pyralog |
| [IMPLEMENTATION_PLAN](IMPLEMENTATION_PLAN.md) | Complete roadmap (6 phases, 16-23 weeks) |
| [RUST_LIBRARIES](RUST_LIBRARIES.md) | Recommended Rust crates |
| [CHANGELOG](CHANGELOG.md) | Version history and releases |
| [PROJECT_SUMMARY](PROJECT_SUMMARY.md) | Complete project overview |
| [BRANDING](BRANDING.md) 🎨 | Brand identity (Egyptian theme) |

### 📝 Blog Series
| Post | Title | Topics |
|------|-------|--------|
| [01](blog/01-introducing-pyralog.md) | Introducing Pyralog | Platform overview |
| [02](blog/02-obelisk-sequencer.md) | Obelisk Sequencer | Novel persistent atomic primitive |
| [03](blog/03-pharaoh-network.md) | Pharaoh Network | Eliminating bottlenecks |
| [04](blog/04-28-billion-ops.md) | 28 Billion Ops/Sec | Architectural deep-dive |
| [05](blog/05-rust-infrastructure.md) | Building in Rust | Lessons learned |
| [06](blog/06-cryptographic-verification.md) | Cryptographic Verification | Merkle trees, BLAKE3 |
| [07](blog/07-multi-model-database.md) | Multi-Model Database | Five data models |
| [08](blog/08-batuta-language.md) | Batuta Language | Actor-first programming |
| [09](blog/09-actor-concurrency.md) | Actor Concurrency | Supervision trees |
| [10](blog/10-wireguard-networking.md) | WireGuard Networking | Quantum resistance |
| [11](blog/11-zero-copy-data-flow.md) | Zero-Copy Data Flow | Arrow IPC, mmap, DMA |
| [12](blog/12-shen-ring.md) | Shen Ring Architecture | Five distributed patterns |
| [13](blog/13-perfect-hash-maps.md) | Perfect Hash Maps | PPHM algorithm |
| [14](blog/14-deduplication.md) | Multi-Layer Deduplication | Five strategies |
| [15](blog/15-memory-only.md) | Memory-Only Mode | Ultra-fast ephemeral storage |
| [16](blog/16-five-interfaces.md) | Five Query Interfaces | SQL, JSON-RPC, GraphQL, PRQL, Batuta |
| [17](blog/17-batuta-modes.md) | Batuta Execution Modes | Client-side vs server-side |
| [18](blog/18-category-theory.md) | Category Theory | Practical applications |
| [19](blog/19-tensor-database.md) | Tensor Database | ML models as first-class citizens |
| [20](blog/20-lsm-arrow.md) | LSM Trees Meet Arrow | Hybrid storage |
| [21](blog/21-decentralized.md) | Decentralized Network | Cluster to Network |
| [22](blog/22-zk-proofs.md) | Zero-Knowledge Proofs | SNARKs vs STARKs |
| [23](blog/23-pow-useful.md) | PoW Without Miners | Useful computation |
| [24](blog/24-operations.md) | Production Operations | Deployment, monitoring |
| [25](blog/25-kafka-migration.md) | Migrating from Kafka | Zero-downtime strategy |
| [26](blog/26-event-driven.md) | Event-Driven Architecture | Event sourcing, CQRS, CDC |
| [27](blog/27-analytics.md) | Real-Time Analytics | vs ClickHouse |
| [28](blog/28-graphmd.md) | Building in Public | GraphMD workflow |
| [29](blog/29-shared-nothing.md) | Shared-Nothing Architecture | Actor model, worker pools |
| [30](blog/30-sulise.md) | Sulise Toolkit | Language development theory |

**Series Statistics**: 30 posts, ~150K words, ~9 hours reading time

---

## 🎯 Quick Start Paths

### I want to learn Pyralog
```
1. README → Overview and features
2. CORE_CONCEPTS → Fundamentals
3. ARCHITECTURE → System design
4. PAPER → Novel contributions
5. Blog Series → Practical insights
```

### I want to deploy Pyralog
```
1. QUICK_START → Basic setup
2. OPERATIONS → Production deployment
3. PERFORMANCE → Tuning guide
4. COMPARISON → vs alternatives
5. FAQ → Common issues
```

### I want to develop with Pyralog
```
1. QUICK_START → Setup
2. EXAMPLES → Code patterns
3. BATUTA → Programming language
4. PRQL/GraphQL → Query languages
5. API docs → Reference
```

### I want to contribute
```
1. CONTRIBUTING → Guidelines
2. ARCHITECTURE → Internals
3. IMPLEMENTATION_PLAN → Roadmap
4. RUST_LIBRARIES → Ecosystem
5. CHANGELOG → Development status
```

### I want to migrate from Kafka
```
1. COMPARISON → Differences
2. FAQ → Compatibility
3. blog/25-kafka-migration → Migration guide
4. OPERATIONS → Deployment
```

### I want ML/AI features
```
1. TENSOR_DATABASE → Native tensor support
2. ARROW → Columnar data format
3. DATA_FORMATS → Safetensors, DLPack
4. blog/19-tensor-database → Practical guide
```

### I want cryptographic verification
```
1. CRYPTOGRAPHIC_VERIFICATION → Complete guide
2. WIREGUARD_PROTOCOL → Secure networking
3. blog/06-cryptographic-verification → Deep dive
4. DECENTRALIZED → Blockchain-style features
```

### I want multi-model database
```
1. MULTI_MODEL_DATABASE → Five data models
2. ARROW → Unified storage
3. BATUTA → Query language
4. blog/07-multi-model-database → Examples
```

### I want decentralized systems
```
1. DECENTRALIZED → Cluster vs Network
2. DADBS → Autonomous systems
3. blog/21-decentralized → Practical guide
4. blog/22-zk-proofs → Zero-knowledge proofs
```

### I want actor-based concurrency
```
1. ACTOR_MODEL → Theory and practice
2. BATUTA → Actor-first language
3. blog/09-actor-concurrency → Deep dive
4. blog/29-shared-nothing → Architecture patterns
```

### I want theoretical foundations
```
1. BATUTA → Category Theory, FRA
2. FUNCTIONAL_RELATIONAL_ALGEBRA → Pure functional queries
3. ACTOR_MODEL → Formal semantics
4. blog/18-category-theory → Practical applications
5. blog/30-sulise → Language theory
```

---

## 📊 Documentation Statistics

**Total Documentation**: 144 files, 93,966 lines, 328,018 words, ~27 hours reading time

### By Category
| Category | Files | Lines | Words | Reading Time |
|----------|-------|-------|-------|--------------|
| Core Documentation | 48 | 66,654 | 217,508 | ~18 hours |
| Blog Posts | 34 | 21,080 | 67,641 | ~5.6 hours |
| Diagrams | 11 | 1,158 | - | Visual |
| Sulise Docs | 60 | 5,992 | 41,980 | ~3.5 hours |

### Highlights
- **6.3× more docs than Kafka**
- **11.8× more than Redis**
- **1.9× more than PostgreSQL**
- **30 blog posts** (~150K words)
- **10 architecture diagrams**
- **Complete API coverage**
- **All query languages documented**

### Growth Timeline
```
October 2025:     ~110,000 words (Phase 1: Initial docs + blog series 1-10)
November 2025:    ~328,000 words (Phase 2: Expansion series 11-30 + comprehensive docs)

Growth: 3× increase in 1 month
```

---

## 📋 Documentation Coverage

### ✅ Complete Documentation
- Core architecture and design
- Novel contributions (Obelisk Sequencer, Pharaoh Network, PPHM)
- Actor model with formal semantics
- Tensor database for ML/AI
- Multi-model database with Category Theory
- Cryptographic verification and zero-trust
- Decentralized systems (PoW, PoS, zk-proofs)
- Storage layer (LSM-Tree, hybrid, memory-only)
- Query languages (Batuta, PRQL, GraphQL, JSON-RPC/WS)
- WireGuard protocol with quantum resistance
- Operations and performance
- Comparison with alternatives
- Complete blog series (30 posts)
- Visual diagrams (10 Mermaid diagrams)

---

## 🔍 Finding Information

### Search Strategies
1. **By keyword**: Search across all `.md` files in your editor
2. **By topic**: Use the tables above to find relevant documents
3. **By question**: Check [FAQ.md](FAQ.md) first
4. **By use case**: See "Quick Start Paths" above
5. **Visual learner?**: Start with [diagrams/](diagrams/)

### Common Topics Reference
| Topic | Primary Document | Related Documents |
|-------|------------------|-------------------|
| Research contributions | [PAPER.md](PAPER.md) | ARCHITECTURE, DESIGN |
| Installation | [QUICK_START.md](QUICK_START.md) | OPERATIONS |
| Architecture | [ARCHITECTURE.md](ARCHITECTURE.md) | DESIGN, PAPER |
| Performance | [PERFORMANCE.md](PERFORMANCE.md) | OPERATIONS |
| Comparison | [COMPARISON.md](COMPARISON.md) | FAQ, TIKV_COMPARISON |
| Examples | [EXAMPLES.md](EXAMPLES.md) | QUICK_START |
| Programming | [BATUTA.md](BATUTA.md) | PRQL, GRAPHQL |
| Queries | [PRQL.md](PRQL.md) | BATUTA, GRAPHQL |
| Storage | [STORAGE.md](STORAGE.md) | ARROW, DATA_FORMATS |
| ML/AI | [TENSOR_DATABASE.md](TENSOR_DATABASE.md) | ARROW, DATA_FORMATS |
| Security | [CRYPTOGRAPHIC_VERIFICATION.md](CRYPTOGRAPHIC_VERIFICATION.md) | WIREGUARD_PROTOCOL |
| Decentralized | [DECENTRALIZED.md](DECENTRALIZED.md) | DADBS |
| Actors | [ACTOR_MODEL.md](ACTOR_MODEL.md) | BATUTA |

---

## 🆘 Getting Help

### Support Channels
1. **Documentation**: Search this index and linked documents
2. **FAQ**: [FAQ.md](FAQ.md) for common questions
3. **Community**:
   - GitHub Discussions (design, features, questions)
   - Discord Server (real-time chat, support)
   - GitHub Issues (bug reports only)
4. **Email**: dev@pyralog.io
5. **Blog**: Read the [30-post blog series](blog/README.md) for deep dives

### Troubleshooting
- **Installation issues?** → [QUICK_START.md](QUICK_START.md), [OPERATIONS.md](OPERATIONS.md)
- **Performance issues?** → [PERFORMANCE.md](PERFORMANCE.md), [FAQ.md](FAQ.md)
- **Architecture questions?** → [ARCHITECTURE.md](ARCHITECTURE.md), [PAPER.md](PAPER.md)
- **Migration help?** → [COMPARISON.md](COMPARISON.md), [blog/25-kafka-migration.md](blog/25-kafka-migration.md)

---

## 📚 External Resources

### Rust Learning
- [The Rust Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)

### Distributed Systems
- [Raft Paper](https://raft.github.io/raft.pdf) - Consensus algorithm
- [Designing Data-Intensive Applications](https://dataintensive.net/) - Essential reading
- [LogDevice Paper](https://engineering.fb.com/2017/08/31/core-infra/logdevice-a-distributed-data-store-for-logs/) - Facebook's log store

### Pyralog Ecosystem
- [shared-nothing](https://github.com/pyralog/shared-nothing) - Actor model, worker pools, high-performance message passing (~80ns latency, 12M msg/sec)
- [GraphMD](https://github.com/graphmd-lpe/graphmd) - Literate Programming Environment for AI-assisted development

### Related Projects
- [Redpanda](https://redpanda.com/) - Kafka-compatible in C++
- [Apache Kafka](https://kafka.apache.org/) - Industry standard
- [LogDevice](https://logdevice.io/) - Facebook's distributed log
- [TiKV](https://tikv.org/) - Distributed key-value with multi-Raft
- [Databend](https://databend.rs/) - Cloud-native data warehouse

---

## 📝 Documentation Quality

All documentation follows these principles:
- ✅ **Clear**: Easy to understand, scannable structure
- ✅ **Complete**: Covers all aspects comprehensively
- ✅ **Accurate**: Technically correct and up-to-date
- ✅ **Current**: Maintained with code changes
- ✅ **Examples**: Practical code samples included
- ✅ **Organized**: Logical structure with cross-references

---

## 🔄 Keeping Documentation Updated

Documentation is version-controlled with code:
- Updated with each feature or change
- Reviewed in all pull requests
- Versioned with releases
- Community contributions welcome

**Want to improve the docs?** See [CONTRIBUTING.md](CONTRIBUTING.md)

---

**Last Updated**: 2025-11-03 (includes all query languages, storage documentation, complete blog series)

**Maintainers**: Pyralog Team

**License**: MIT-0 (code) & CC0-1.0 (documentation)

*Found an issue? [Open an issue](https://github.com/pyralog/pyralog/issues) or submit a PR!*
