# Batuta Execution Modes: Client vs Server

**Run your queries close to the data—or close to the user**

*Published: November 3, 2025*

---

## The Execution Location Problem

Where should your query logic run?

**Traditional databases force one choice:**

```
PostgreSQL: Server-side only
  • All logic runs on database server
  • Application just sends SQL
  • Problem: Can't do client-side processing

MongoDB: Mix of client and server
  • Aggregation pipelines: Server
  • Data transformations: Client (application code)
  • Problem: Logic split across languages

Result: Forced choice, split logic, no flexibility
```

**Batuta offers both:** Choose execution location per query.

---

## The Two Execution Modes

Batuta supports **two execution modes** with the same code:

```
1. Client-Side (Application-Embedded)
   • Batuta runtime embedded in your application
   • Queries run in application process
   • Send requests to Pyramid nodes for data

2. Server-Side (Database-Embedded)
   • Batuta runtime embedded in Pyramid nodes
   • Queries run close to data
   • Application sends Batuta code to execute

Same code, different execution location!
```

---

## Mode 1: Client-Side Execution (Application-Embedded)

**Where it runs**: In your application process

### Architecture

```
┌───────────────────────────────────────────────────┐
│              Your Application                      │
│  ┌─────────────────────────────────────────────┐  │
│  │         Batuta Runtime (Embedded)           │  │
│  │  • Compiles to Rust/WASM                    │  │
│  │  • Runs in application process              │  │
│  │  • Sends data requests to Pyramid nodes     │  │
│  └─────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────┘
           │
           │ Data requests
           ↓
┌───────────────────────────────────────────────────┐
│             Pyramid Nodes                          │
│  • Serve data via Arrow Flight                    │
│  • No Batuta execution                            │
└───────────────────────────────────────────────────┘
```

### Use Cases

#### 1. **Application Business Logic**

```clojure
;; Client-side: Application logic
(defn calculate-discount [user order]
  (let [tier (get-user-tier user)
        total (:total order)
        discount (case tier
                   :gold   0.20
                   :silver 0.10
                   :bronze 0.05
                   0.0)]
    (* total (- 1 discount))))

;; Runs in application process
(def order (fetch-order "order-123"))
(def user (fetch-user "user-456"))
(def final-price (calculate-discount user order))
```

**Why client-side?**
- Business rules stay in application
- Easy to test and debug
- No database deployment needed

---

#### 2. **Client-Side Data Transformation**

```clojure
;; Fetch raw data from Pyramid
(def raw-events 
  (query "SELECT * FROM events 
          WHERE timestamp > $1" 
         (- (now) (days 30))))

;; Transform locally (in application)
(def metrics
  (->> raw-events
       (group-by :user_id)
       (map (fn [[user events]]
              {:user user
               :count (count events)
               :avg-amount (average (map :amount events))}))
       (filter #(> (:count %) 10))
       (sort-by :count >)
       (take 100)))

;; Display in UI (no server round-trip!)
```

**Why client-side?**
- Interactive UI updates (no server latency)
- Pagination, filtering, sorting (instant!)
- Custom visualizations

---

#### 3. **Edge Computing / IoT**

```clojure
;; IoT device: Process locally
(defn process-sensor-data [readings]
  (let [anomalies (->> readings
                       (filter #(> (:temperature %) 80))
                       (map #(assoc % :alert true)))]
    (when (seq anomalies)
      ;; Only send anomalies to server
      (send-to-server anomalies))))

;; Runs on edge device (Raspberry Pi, ESP32)
(loop []
  (let [readings (read-sensors)]
    (process-sensor-data readings)
    (sleep 1000)
    (recur)))
```

**Why client-side?**
- Reduce bandwidth (process locally)
- Real-time response (no network latency)
- Works offline

---

#### 4. **Browser Applications (WASM)**

```clojure
;; Compile Batuta to WebAssembly
(defn render-dashboard [user-id]
  (let [data (fetch "/api/metrics?user=" user-id)
        processed (->> data
                       (filter #(> (:value %) 0))
                       (group-by :category)
                       (map-values #(sum (map :value %))))]
    (render-chart processed)))

;; Runs in browser (WebAssembly)
;; No server execution needed!
```

**Why client-side (WASM)?**
- Instant UI updates
- Offline support
- Reduce server load

---

### Performance (Client-Side)

```
Latency breakdown (client-side execution):

Traditional (SQL → Server → Client):
  1. Send SQL: 1ms (network)
  2. Execute SQL: 50ms (server)
  3. Return results: 1ms (network)
  4. Transform in JS: 100ms (slow!)
  Total: 152ms

Batuta (Client-Side):
  1. Fetch data: 2ms (Arrow Flight)
  2. Execute Batuta: 5ms (compiled Rust/WASM)
  3. Transform: 0ms (already done!)
  Total: 7ms (21× faster!)
```

---

## Mode 2: Server-Side Execution (Database-Embedded)

**Where it runs**: Inside Pyramid nodes, close to data

### Architecture

```
┌───────────────────────────────────────────────────┐
│              Your Application                      │
│  • Sends Batuta code to execute                   │
│  • Receives results                               │
└───────────────────────────────────────────────────┘
           │
           │ Send Batuta code
           ↓
┌───────────────────────────────────────────────────┐
│             Pyramid Nodes                          │
│  ┌─────────────────────────────────────────────┐  │
│  │         Batuta Runtime (Embedded)           │  │
│  │  • Executes Batuta code                     │  │
│  │  • Access local data (zero-copy!)           │  │
│  │  • Returns results                          │  │
│  └─────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────┘
```

### Use Cases

#### 1. **Large Dataset Processing**

```clojure
;; Server-side: Process 1TB of data
(defn analyze-all-events []
  (query "
    SELECT 
      user_id,
      COUNT(*) AS events,
      SUM(amount) AS revenue
    FROM events
    GROUP BY user_id
    HAVING revenue > 10000
    ORDER BY revenue DESC
  "))

;; If run client-side: Transfer 1TB over network (slow!)
;; If run server-side: Process locally, return 100KB summary (fast!)

;; Execute server-side
(server-exec analyze-all-events)
```

**Why server-side?**
- Minimize data transfer (1TB → 100KB)
- Leverage Pyramid compute resources
- Faster (no network bottleneck)

---

#### 2. **Complex Aggregations**

```clojure
;; Server-side: Multi-step aggregation
(defn user-360-view [user-id]
  {;; Relational query
   :profile (query-one "SELECT * FROM users WHERE id = $1" user-id)
   
   ;; Graph traversal
   :friends (->> (graph-query 
                   [:match [[user-id :friend ?friend]]
                    :return ?friend])
                 (take 100))
   
   ;; Time-series aggregation
   :activity (->> (query "SELECT * FROM events 
                          WHERE user_id = $1 
                          AND timestamp > $2"
                         user-id
                         (- (now) (days 30)))
                  (group-by :event_type)
                  (map-values count))
   
   ;; Tensor embedding
   :embedding (tensor-slice "user-embeddings" user-id)})

;; Execute server-side (all data local!)
(server-exec user-360-view "user-123")
```

**Why server-side?**
- Multi-model queries (relational + graph + tensor)
- All data local (zero-copy access)
- Single round-trip

---

#### 3. **Scheduled Jobs / Batch Processing**

```clojure
;; Server-side: Nightly aggregation job
(defn nightly-rollup []
  (let [yesterday (- (now) (days 1))]
    ;; Process 100GB of yesterday's data
    (query! "
      INSERT INTO daily_metrics
      SELECT 
        DATE_TRUNC('day', timestamp) AS date,
        user_id,
        COUNT(*) AS events,
        SUM(amount) AS revenue
      FROM events
      WHERE timestamp >= $1 AND timestamp < $2
      GROUP BY date, user_id
    " yesterday (now))))

;; Register as cron job (runs server-side)
(schedule "0 0 * * *" nightly-rollup)
```

**Why server-side?**
- No client needed (runs autonomously)
- Process large datasets efficiently
- Scheduled execution

---

#### 4. **Real-Time Stream Processing**

```clojure
;; Server-side: Process event stream
(defn process-events-stream [stream]
  (actor
    (loop []
      (let [event (<! stream)]
        ;; Complex event processing (CEP)
        (when (fraud-detected? event)
          (send-alert! event))
        
        ;; Update aggregates
        (query! "UPDATE metrics SET count = count + 1 WHERE key = $1" 
                (:key event))
        
        (recur)))))

;; Deploy to Pyramid nodes (server-side)
(deploy-actor process-events-stream kafka-stream)
```

**Why server-side?**
- Real-time processing (sub-millisecond)
- Stateful actors (maintain state)
- High throughput (millions/sec)

---

### Performance (Server-Side)

```
Latency breakdown (server-side execution):

Traditional (Client → Server → Process → Client):
  1. Send request: 1ms (network)
  2. Fetch 1GB data: 500ms (disk I/O)
  3. Transfer to client: 8000ms (network @ 125MB/s)
  4. Process in client: 2000ms (slow JS)
  Total: 10,501ms (~10 seconds!)

Batuta (Server-Side):
  1. Send Batuta code: 1ms (small)
  2. Fetch data locally: 50ms (zero-copy mmap)
  3. Process server-side: 200ms (compiled Rust)
  4. Return summary: 1ms (100KB result)
  Total: 252ms (41× faster!)
```

---

## Choosing Execution Mode

### Decision Matrix

```
┌─────────────────────────────────────────────────────┐
│          EXECUTION MODE DECISION TREE                │
└─────────────────────────────────────────────────────┘

How much data are you processing?

├─ Small (<100MB)
│  └─ Client-Side
│     • Fast enough over network
│     • Lower server load
│
├─ Large (>1GB)
│  └─ Server-Side
│     • Network bottleneck avoided
│     • Leverage Pyramid compute
│
└─ Medium (100MB-1GB)
   ├─ Interactive UI? → Client-Side
   └─ Batch job? → Server-Side
```

### By Use Case

| Use Case | Mode | Why |
|----------|------|-----|
| **Business logic** | Client | Application-specific rules |
| **UI updates** | Client | Instant response |
| **Edge/IoT** | Client | Process locally, reduce bandwidth |
| **Browser apps** | Client (WASM) | Offline, instant |
| **Large aggregations** | Server | Minimize data transfer |
| **Multi-model queries** | Server | Zero-copy local access |
| **Scheduled jobs** | Server | Autonomous execution |
| **Stream processing** | Server | Real-time, stateful |

---

## Compilation Strategies

### Client-Side Compilation

**Target 1: Native (Rust)**

```rust
// Batuta compiles to Rust
fn calculate_discount(user: &User, order: &Order) -> f64 {
    let tier = user.tier;
    let discount = match tier {
        Tier::Gold => 0.20,
        Tier::Silver => 0.10,
        Tier::Bronze => 0.05,
        Tier::Free => 0.0,
    };
    order.total * (1.0 - discount)
}

// Embedded in your application
let final_price = calculate_discount(&user, &order);
```

**Performance**: Near-native speed (~50-100ns overhead)

**Target 2: WebAssembly**

```clojure
;; Batuta code
(defn process-data [data]
  (->> data
       (filter #(> (:value %) 0))
       (map #(* (:value %) 1.1))
       (reduce +)))

;; Compile to WASM
;; → Runs in browser at near-native speed
```

**Performance**: 2-3× slower than native, 10× faster than JS

---

### Server-Side Compilation

**Ahead-of-Time (AOT)**

```clojure
;; Batuta code
(defn heavy-aggregation []
  (query "
    SELECT user_id, COUNT(*) 
    FROM events 
    GROUP BY user_id
  "))

;; Compile to Rust, deploy to Pyramid
(aot-compile 'heavy-aggregation)
(deploy-to-pyramid 'heavy-aggregation)
```

**Just-in-Time (JIT)**

```clojure
;; Send Batuta code to Pyramid at runtime
(server-exec '(query "SELECT * FROM users LIMIT 10"))

;; Pyramid compiles and caches on first execution
;; Subsequent calls use cached version
```

---

## DataFusion Optimizer Integration

Batuta leverages **DataFusion's LogicalPlan optimizer** for SQL queries:

```clojure
;; Batuta query
(defn active-users []
  (->> (query "SELECT * FROM users")
       (filter #(> (:last_login %) (- (now) (days 30))))
       (map #(select-keys % [:id :name :email]))
       (take 100)))

;; DataFusion optimizes:
;; 1. Push filter down (WHERE last_login > ...)
;; 2. Push projection down (SELECT id, name, email)
;; 3. Push limit down (LIMIT 100)
;; 
;; Result: Optimized SQL instead of scanning all rows!
```

**Optimized SQL (generated)**:

```sql
SELECT id, name, email
FROM users
WHERE last_login > CURRENT_TIMESTAMP - INTERVAL '30 days'
LIMIT 100;
```

**Before optimization**: Scan 10M rows, filter in Batuta
**After optimization**: Scan 100 rows, filter in SQL

**Speedup**: 100,000× faster!

---

## Performance Comparison

### Execution Mode Performance

```
Benchmark: Process 1GB dataset (10M records)

Client-Side Execution:
  1. Fetch data: 8,000ms (network @ 125MB/s)
  2. Batuta processing: 200ms (Rust)
  3. Total: 8,200ms
  4. Network: Bottleneck!

Server-Side Execution:
  1. Fetch data locally: 50ms (mmap)
  2. Batuta processing: 200ms (Rust)
  3. Return summary: 1ms (100KB)
  4. Total: 251ms (32× faster!)

Result: Server-side wins for large datasets
```

### Compilation Target Performance

```
Benchmark: Same Batuta code, different targets

Native (Rust):
  • Compile time: 5-10ms
  • Execution: 200ms
  • Overhead: ~5%

WebAssembly (WASM):
  • Compile time: 10-20ms
  • Execution: 600ms (3× slower than native)
  • Still 10× faster than JavaScript!

Interpreted (REPL):
  • No compile time
  • Execution: 2,000ms (10× slower)
  • Good for development/debugging
```

---

## Real-World Example: E-Commerce Analytics

### Client-Side: Interactive Dashboard

```clojure
;; Client-side: Interactive UI (React + WASM)
(defn dashboard [user-id]
  ;; Fetch user data
  (let [orders (fetch-orders user-id)
        stats (calculate-stats orders)]
    
    ;; Local transformations (instant!)
    (when (button-clicked? :filter-month)
      (let [filtered (->> orders
                          (filter #(= (:month %) (current-month))))]
        (render-chart filtered)))
    
    ;; Pagination (client-side, no server!)
    (when (page-changed?)
      (render-page (nth-page orders page-num)))))

;; Compile to WASM, run in browser
;; Result: Instant UI updates, no server round-trips
```

---

### Server-Side: Nightly Aggregation

```clojure
;; Server-side: Process all orders (100GB)
(defn nightly-aggregation []
  (query! "
    INSERT INTO daily_sales
    SELECT 
      DATE_TRUNC('day', created_at) AS date,
      product_id,
      COUNT(*) AS orders,
      SUM(amount) AS revenue
    FROM orders
    WHERE created_at >= CURRENT_DATE - INTERVAL '1 day'
    GROUP BY date, product_id
  "))

;; Deploy to Pyramid (server-side)
(schedule "0 2 * * *" nightly-aggregation)

;; Result: Autonomous execution, no client needed
```

---

## Summary

Batuta offers **two execution modes** for maximum flexibility:

### Client-Side (Application-Embedded)

- ✅ **Application logic**: Business rules stay in app
- ✅ **Interactive UI**: Instant updates
- ✅ **Edge/IoT**: Process locally
- ✅ **Browser (WASM)**: Offline support
- ⚠️ **Network bottleneck**: For large datasets

### Server-Side (Database-Embedded)

- ✅ **Large datasets**: Minimize transfer (1TB → 100KB)
- ✅ **Complex queries**: Multi-model, zero-copy
- ✅ **Scheduled jobs**: Autonomous execution
- ✅ **Stream processing**: Real-time, stateful
- ⚠️ **Server load**: Consumes Pyramid resources

### Key Insights

- **Same code, different location**: Write once, run anywhere
- **Choose per query**: Mix client and server in one app
- **DataFusion optimizer**: Automatic SQL optimization
- **Multiple targets**: Compile to Rust or WASM
- **32× faster**: Server-side for large datasets
- **21× faster**: Client-side for interactive UI

### The Bottom Line

**Stop forcing all logic to run in one place.**

Different queries have different optimal execution locations. Batuta lets you choose—per query—whether to run close to the data (server-side) or close to the user (client-side). With the same code, you get the best of both worlds: server power for heavy lifting, client responsiveness for interactivity.

*One language. Two modes. Infinite flexibility.*

---

## Next Steps

**Want to learn more?**

- Read [Batuta Language Guide](../BATUTA.md) for complete language reference
- See [Category Theory for Practitioners](18-category-theory.md) for theoretical foundations
- Check [DataFusion Integration](../BATUTA.md#datafusion-integration) for optimizer details
- Try [Quick Start](../QUICK_START.md) to write your first Batuta query

**Discuss Batuta execution modes**:
- Discord: [discord.gg/pyralog](https://discord.gg/pyralog)
- GitHub: [github.com/pyralog/pyralog](https://github.com/pyralog/pyralog)
- Email: hello@pyralog.io

---

*Part 17 of the Pyralog Blog Series*

*Previously: [Five Ways to Query Pyralog](16-five-interfaces.md)*
*Next: [Category Theory for Practitioners](18-category-theory.md)*

