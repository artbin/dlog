# Building in Public with GraphMD: 6-Phase LLM Workflow

**How 77K lines of documentation were created in 6 weeks**

*Published: November 3, 2025*

---

## The Challenge

**Goal**: Build Pyralog—a distributed database with:
- Novel Obelisk Sequencer architecture
- Hybrid LSM + Arrow storage
- Multi-model support
- Tensor database
- Zero-knowledge proofs
- Multiple query languages

**Timeline**: 6 weeks
**Team**: 1 developer + Claude Sonnet 4.5
**Method**: GraphMD workflow

---

## What is GraphMD?

**GraphMD**: Literate Programming Environment for Markdown-Based Executable Knowledge Graphs

```
Traditional development:
  Code → Comments → (Maybe) Docs

GraphMD workflow:
  Docs → Architecture → Code
  
Philosophy: Documentation-first, AI-assisted
```

**Repository**: https://github.com/graphmd-lpe/graphmd

---

## The 6-Phase Workflow

```
┌──────────────────────────────────────────────┐
│          GRAPHMD WORKFLOW (6 PHASES)          │
├──────────────────────────────────────────────┤
│                                              │
│  Phase 1: Requirements (Week 1)             │
│  └─ Brain dump → Structured docs            │
│                                              │
│  Phase 2: Architecture (Week 2)             │
│  └─ System design → Diagrams                │
│                                              │
│  Phase 3: Documentation (Week 3-4)          │
│  └─ Detailed specs → API docs               │
│                                              │
│  Phase 4: Code Generation (Week 5)          │
│  └─ Docs → Implementation                   │
│                                              │
│  Phase 5: Testing & Validation (Week 6)     │
│  └─ Verify → Iterate                        │
│                                              │
│  Phase 6: Publication (Ongoing)             │
│  └─ Blog posts, tutorials, examples         │
│                                              │
└──────────────────────────────────────────────┘
```

---

## Phase 1: Requirements Gathering

### Week 1: Brain Dump

**Process**:
```
Day 1-2: High-level vision
  • "I want a distributed log like Kafka, but faster"
  • "Need exactly-once semantics"
  • "Multi-model support"
  
Day 3-4: Detailed requirements
  • Performance targets (p99 < 1ms)
  • Storage targets (1PB scale)
  • API requirements (SQL, DataFrame, custom DSL)
  
Day 5-7: Prioritization
  • Must-have: Distributed log, Raft consensus
  • Should-have: Multi-model, tensor support
  • Nice-to-have: Zero-knowledge proofs
```

**Output**: `REQUIREMENTS.md` (5,000 words)

---

### LLM Assistance

```markdown
Prompt: "I want to build a distributed database. Here are my goals:
- 500M writes/sec
- p99 < 1ms latency
- Strong consistency
- Multi-model support

Help me structure the requirements."

LLM Response: [Structured requirements doc with sections for:
- Functional requirements
- Non-functional requirements  
- Performance targets
- API requirements
- Constraints
]
```

**LLM role**: Structure, organize, identify gaps

---

## Phase 2: Architecture Design

### Week 2: System Design

**Process**:
```
Day 1-2: High-level architecture
  • Two-tier: Obelisk (coordination) + Pyramid (storage)
  • Pharaoh Network for ID generation
  • Raft for consensus
  
Day 3-4: Component design
  • LSM-Tree storage
  • PPHM indexes
  • Arrow columnar format
  
Day 5-7: Integration points
  • How do components interact?
  • Data flow diagrams
  • API boundaries
```

**Output**: 
- `BRANDING.md` (Egyptian theme)
- `NODES.md` (Obelisk + Pyramid)
- `SHEN_RING.md` (Distributed patterns)
- `diagrams/system-architecture.mmd`

---

### LLM Assistance

```markdown
Prompt: "Design a two-tier architecture for distributed coordination.
Requirements:
- Lightweight coordination nodes (Obelisk)
- Heavy storage nodes (Pyramid)
- No single point of failure

Suggest an architecture."

LLM Response: [Detailed architecture with:
- Obelisk nodes: Sparse file counters, no consensus
- Pyramid nodes: LSM storage, Raft per partition
- Pharaoh Network: Gossip for cluster membership
]
```

**LLM role**: Design alternatives, identify trade-offs

---

## Phase 3: Documentation Writing

### Week 3-4: Detailed Specifications

**Process**:
```
Week 3:
  • Storage layer (STORAGE.md, 1,500 lines)
  • Arrow integration (ARROW.md, 1,400 lines)
  • PPHM algorithm (PPHM.md, 800 lines)
  • Deduplication (DEDUPLICATION.md, 600 lines)
  
Week 4:
  • Tensor database (TENSOR_DATABASE.md, 900 lines)
  • Data formats (DATA_FORMATS.md, 800 lines)
  • Query languages (PRQL.md, 1,400 lines)
  • APIs (GRAPHQL.md, 1,400 lines, JSONRPC_WEBSOCKET.md, 1,300 lines)
```

**Total**: 25 documents, 35,000 lines

---

### LLM Assistance

```markdown
Prompt: "Document the LSM-Tree storage architecture for Pyralog.
Include:
- Why LSM-Tree (write optimization)
- Architecture (MemTable, L0, L1+)
- Compaction strategies
- Hybrid storage (native + external files)

Make it detailed with code examples."

LLM Response: [STORAGE.md with:
- Complete architecture overview
- Decision matrix (native vs external)
- Rust implementation examples
- Performance characteristics
]
```

**LLM role**: Expand outlines, generate examples, ensure consistency

---

## Phase 4: Code Generation

### Week 5: Implementation

**Process**:
```
Day 1-2: Core data structures
  • AtomicCounter (Obelisk)
  • LSM-Tree (Pyramid)
  • Raft integration
  
Day 3-4: APIs
  • JSON-RPC/WebSocket server
  • Arrow Flight
  • SQL via DataFusion
  
Day 5-7: Integration
  • Wire everything together
  • End-to-end tests
```

**Output**: 15,000 lines of Rust (conceptual, not all implemented yet)

---

### LLM Assistance

```markdown
Prompt: "Implement AtomicCounter using sparse files.
Requirements:
- File size = counter value
- Atomic increment via file truncation
- Memory-mapped for fast reads

Generate Rust code."

LLM Response: [Complete implementation with:
- Struct definition
- Increment method
- Read method
- Error handling
- Tests
]
```

**LLM role**: Generate boilerplate, suggest implementations, write tests

---

## Phase 5: Testing & Validation

### Week 6: Verify Everything

**Process**:
```
Day 1-2: Documentation review
  • Read all 77K lines
  • Check consistency
  • Fix broken links
  
Day 3-4: Architecture validation
  • Does it make sense?
  • Are trade-offs correct?
  • Performance achievable?
  
Day 5-7: Blog series
  • 10 initial blog posts
  • 20 expansion posts (in progress)
```

**Output**: 
- 30 blog posts (150K words)
- Updated documentation index
- Cross-references validated

---

### LLM Assistance

```markdown
Prompt: "Review STORAGE.md for consistency with ARROW.md.
Ensure:
- Same terminology used
- No contradictions
- Links are correct

Report any issues."

LLM Response: [Issues found:
- STORAGE.md says "Arrow Binary blob" but ARROW.md uses "file references"
- Fix: Update STORAGE.md to use consistent terminology
]
```

**LLM role**: Consistency checker, reviewer, editor

---

## Phase 6: Publication

### Ongoing: Share with World

**Channels**:
```
Documentation:
  • GitHub: github.com/pyralog/pyralog
  • Website: pyralog.io (planned)
  
Blog:
  • 30 technical deep-dives
  • 150K+ words
  • 10+ hours read time
  
Community:
  • Discord: discord.gg/pyralog
  • Twitter: @pyralog
  • Reddit: r/pyralog
```

---

## Metrics

### Documentation Stats

```
Total files: 45
Total lines: 77,000
Total words: ~500,000

Breakdown:
  • Core docs (25 files): 35,000 lines
  • Blog posts (30 files): 40,000 lines  
  • Diagrams (10 files): 2,000 lines
```

### Time Spent

```
Phase 1 (Requirements):     20 hours
Phase 2 (Architecture):     30 hours
Phase 3 (Documentation):    80 hours
Phase 4 (Code):            40 hours (ongoing)
Phase 5 (Validation):      30 hours
Phase 6 (Publication):     10 hours

Total: 210 hours over 6 weeks = ~35 hours/week
```

### LLM Token Usage

```
Total tokens: ~50M (context + generation)
Cost: ~$250 (Claude Sonnet 4.5)
Equivalent human hours: ~1,000 hours (at 50K words/hour writing speed)

ROI: 1000 hours / 210 hours = 4.8× productivity gain
```

---

## Key Insights

### 1. Documentation-First Works

```
Traditional: Code → Docs (docs often incomplete)
GraphMD: Docs → Code (code implements spec)

Benefits:
  • Clear requirements before coding
  • Easier collaboration (LLM or human)
  • Better architecture decisions
```

---

### 2. LLM as Co-Author

```
Human strengths:
  • Vision and strategy
  • Novel ideas (Obelisk Sequencer)
  • Trade-off decisions
  
LLM strengths:
  • Structure and organization
  • Code generation
  • Consistency checking
  
Together: 5× faster than solo
```

---

### 3. Iterate in Documentation

```
Bad: Write code, find design flaw, rewrite code
Good: Write docs, find design flaw, rewrite docs

Cost:
  • Rewriting docs: 1 hour
  • Rewriting code: 10 hours
  
Result: 10× cheaper to iterate in docs
```

---

### 4. Build in Public

```
Benefits:
  • Accountability (public commitments)
  • Feedback (community input)
  • Marketing (build audience early)
  • Transparency (trust through openness)
  
Pyralog approach:
  • All docs public on GitHub
  • Blog posts explain decisions
  • Discord for real-time discussion
```

---

## Tools & Technologies

### Core Stack

```
LLM: Claude Sonnet 4.5 via Cursor
Version control: Git + GitHub
Diagrams: Mermaid
Documentation: Markdown
Code examples: Rust
```

### GraphMD Features

```
1. Knowledge graphs: Linked Markdown files
2. Literate programming: Docs + code interleaved
3. LLM integration: Prompts embedded in docs
4. Version tracking: Git for docs + code
5. Cross-references: Auto-validate links
```

---

## Lessons for Your Project

### Do's

✅ **Start with docs, not code**
- Write README first
- Design architecture in Markdown
- Spec APIs before implementing

✅ **Use LLM as collaborator**
- Structure ideas → LLM organizes
- Outline → LLM expands
- Draft → LLM refines

✅ **Iterate cheaply**
- Find design flaws in docs
- Fix before coding
- Validate with LLM

✅ **Build in public**
- Share early, share often
- Get feedback fast
- Build community

---

### Don'ts

❌ **Don't skip docs**
- "I'll document later" = never
- Docs-first forces clarity

❌ **Don't blindly trust LLM**
- Verify technical claims
- Test code examples
- Review for consistency

❌ **Don't work in isolation**
- Share docs publicly
- Ask for feedback
- Incorporate suggestions

❌ **Don't fear incompleteness**
- Ship docs before code
- Iterate based on feedback
- Perfect is enemy of good

---

## Summary

**GraphMD workflow** enabled building Pyralog in 6 weeks:

### Process
1. **Requirements**: Brain dump → Structured docs
2. **Architecture**: Design → Diagrams
3. **Documentation**: Specs → Examples (77K lines)
4. **Code**: Docs → Implementation
5. **Validation**: Review → Iterate
6. **Publication**: Blog → Community

### Results
- 77K lines of documentation
- 30 blog posts (150K words)
- 45 files (requirements, architecture, APIs, tutorials)
- 10 architecture diagrams

### Productivity
- **5× faster** than solo human
- **10× cheaper** to iterate in docs than code
- **4.8× ROI** from LLM assistance ($250 → 1,000 hours saved)

### The Bottom Line

**Documentation-first + LLM-assisted = shipping fast with high quality**. Pyralog went from concept to comprehensive documentation in 6 weeks using GraphMD. The code is catching up, but the foundation is solid.

*Document first. Ship faster.*

---

## Next Steps

- Try [GraphMD](https://github.com/graphmd-lpe/graphmd)
- Read [Pyralog Documentation](https://github.com/pyralog/pyralog)
- Join [Discord](https://discord.gg/pyralog) to discuss

---

*Part 28 of the Pyralog Blog Series*

*Previously: [Real-Time Analytics](27-analytics.md)*
*Next: [Shared-Nothing Architecture](29-shared-nothing.md)*

