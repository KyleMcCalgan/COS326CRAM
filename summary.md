# COS 326 — New Stuff Summary

Lectures 20–24 condensed: NewSQL · Big Data technologies · Data mining · Data warehousing & OLAP (1 & 2). Source: the five lecture note files. `L20` `L21` `L22` `L23` `L24`

⏱ Total: 1 h 04 min to study · about 28 min to skim

## Big picture & how the lectures connect

- `L20` **NewSQL** — Run transactions (OLTP) with SQL + ACID, scaled out.
- `L21` **Big Data tech** — Store & process huge data: Hadoop → Spark → cloud/streaming.
- `L22` **Data mining** — Turn stored data into knowledge (rules, classes, clusters).
- `L23` **Warehousing 1** — Where analysis data lives: warehouse, star schema.
- `L24` **Warehousing 2** — Build/load it; SQL OLAP extensions.

- L20 closes the module's data-model story: relational/object-relational (L5–8), XML (L9–10), NoSQL document/graph (L13–19), now NewSQL.
- Split of work: OLTP (L20, operational) vs OLAP (L23–24, analytical) — HTAP (TiDB) tries to do both.
- L21 extends L13 (big data/NoSQL storage) with processing; L22 mining runs on that data, on warehouses via OLAP servers.

## 1 · Transactions & NewSQL `L20`

Agenda: introduction · motivation · ACID & OLTP · key features/architecture · SQL vs NoSQL vs NewSQL · advantages/limitations/trade-offs · examples · applications & database selection.

### 1.1 Introduction & central idea

> **NewSQL** = modern DBMSs for **high-performance OLTP** that keep the **relational model, SQL and ACID** while adding **horizontal scalability** in a modern architecture.

- Problem: traditional relational DBs have mature SQL/transactions but scaling huge transactional loads across servers is architecturally/operationally hard.
- NoSQL response: flexible models, distributed storage, horizontal scaling — but systems differ widely in query languages, data models and transaction guarantees.
- Bank analogy: single big server = safe but limited; NoSQL = scales but weaker guarantees; NewSQL = safe SQL transactions spread over many servers.

### 1.2 ACID & OLTP

| A — Atomicity | C — Consistency | I — Isolation | D — Durability |
| --- | --- | --- | --- |
| Complete entirely or roll back | Integrity constraints preserved | Concurrent transactions behave per isolation guarantees | Committed changes persist despite failures |

**Transfer R500 A→B** (`BEGIN; debit A; credit B; COMMIT;`): crash between debit and credit → rolled back (A); "balance ≥ 0" / money conserved (C); others can't see half-done state (I); survives power loss after commit via logging/replication (D).

- OLTP = operational transactions from apps/users — short, frequent, concurrent. Examples: bank transfers, course registration, online orders, inventory updates, payments/bookings.
- Why it matters: NewSQL must support demanding transactional loads while keeping reliable processing across distributed infrastructure.

|  | OLTP (this lecture) | OLAP (L23–24) |
| --- | --- | --- |
| Workload | Many short reads/writes | Fewer, long, complex reads |
| Example | "Process this order" | "Total sales per region over 5 years" |
| Priority | Correctness + speed per transaction | Throughput over big volumes |

### 1.3 Why NewSQL? Scaling

|  | Vertical (scale up) | Horizontal (scale out) |
| --- | --- | --- |
| Method | Bigger server (CPU/memory/storage) | More nodes; distribute data/processing |
| Limit | Hardware ceiling, cost, architecture | Coordination & network overhead |
| Complexity | Lower (one machine) | Higher: distribution, transaction coordination, network, consistency |
| Failure | Single point of failure | Tolerates node failures (with replication) |

> **Warning:** horizontal scaling does **not** guarantee linear gains — depends on workload, data distribution, network latency and inter-node coordination (4→8 nodes ≠ 2× throughput when transactions span nodes).

**Objective:** keep SQL + reliable relational transactions on distributed, multi-server architectures.

### 1.4 Key features of NewSQL

- 1. SQL & relational management · 2. ACID (even distributed) · 3. Horizontal scalability · 4. Distributed transaction processing (coordinate multi-node txns)
- 5. Partitioning & data distribution · 6. Replication & high availability · 7. Concurrency control (enforces isolation)
- 8. Indexing & query optimisation (indexes cost extra storage, may slow updates) · 9. Crash recovery & durability (logging, replication) · 10. Distributed deployment (servers, data centres, regions)

> **Note:** Not every product implements these the same way — architecture, consistency, replication and scaling behaviour are system-specific.

|  | Partitioning (sharding) | Replication |
| --- | --- | --- |
| What | Data *split* across nodes | Same data *copied* to several nodes |
| Goal | Scalability, parallelism, load spreading | Availability, fault tolerance, recovery |
| Cost | Cross-partition txns need coordination | Copies must stay consistent; extra storage |

Usually combined: e.g. Account table split by account number over 3 nodes, each partition replicated elsewhere. A transfer between accounts on different nodes is a **distributed transaction** — both nodes commit together or both roll back (the main cost of scaling out).

### 1.5 SQL vs NoSQL vs NewSQL

|  | Traditional SQL | NoSQL | NewSQL / distributed SQL |
| --- | --- | --- | --- |
| Data model | Relational tables | Key-value, document, graph, wide-column… | Usually relational tables |
| Query | SQL | System-specific; may be SQL-like/APIs | SQL |
| Schema | Structured, defined | Flexible / schema-dependent | Structured, defined |
| Transactions | ACID common | Varies by product/scope | Designed to retain ACID |
| Scaling | Vertical common (some horizontal) | Horizontal common | Horizontal is a key goal |
| Distribution | Depends on deployment | Common | Designed for distributed deployment |
| Use cases | Enterprise apps, reporting, transactions | Particular data models / distributed access | Transaction-intensive + SQL + scale-out |

- Links: MongoDB (document; find/aggregation) and Neo4j (graph; Cypher) from L13–19 are NoSQL. L13's CAP theorem tension (consistency vs availability) is what NewSQL manages when promising ACID across nodes (CockroachDB favours strong consistency).
- NoSQL varies in models, interfaces and guarantees — not one uniform category.

### 1.6 Advantages & limitations

> **Advantages**
>
> - Familiar SQL & relational structures
> - ACID reliability (banking, orders)
> - Horizontal scalability (add nodes)
> - High availability (replication/fault tolerance)
> - Relational integrity (keys, constraints)
> - Distributed processing of workloads

> **Limitations**
>
> - Operational complexity (monitoring, config, backup, failures)
> - Coordination overhead → latency
> - Network dependency
> - Cost (infrastructure, licensing, networking, expertise)
> - Migration effort (schemas, queries, apps, processes)
> - Workload dependence — not necessarily faster than SQL/NoSQL

### 1.7 Example systems

| System | Standout features | Typical application / scenario |
| --- | --- | --- |
| **VoltDB** | In-memory, distributed, partitioned, very high throughput, Kafka integration, replication/recovery | Telecom usage events + billing, financial transactions |
| **CockroachDB** | Distributed SQL; ACID + strong consistency; serializable isolation; auto data distribution/rebalancing; multi-region | Payments, e-commerce; retailer in several regions |
| **NuoDB** | Scale-out across processes/hosts; continuous availability; cloud + on-premises; geo-distributed | Enterprise app across several sites |
| **TiDB** | Open source; MySQL-compatible; strong consistency; **HTAP** (hybrid transactional + analytical); separated storage/query/cluster components | Retailer processing orders while analysing recent sales |
| **Altibase** | Hybrid in-memory tables + disk tables in one RDBMS (hot data fast, bulk on disk) | Telecom: current operational data + large history |

> **Note:** Altibase only *illustrates* hybrid storage — it doesn't mean all NewSQL systems share a storage design.

### 1.8 Case study — online banking selection

Requirements: millions of accounts, reliable transfers, no inconsistent balances, many concurrent txns, survive server failures, scale, SQL + integrity constraints.

- Why ACID: atomic debit+credit; constraints kept; isolation prevents inconsistent balances; commits survive failures.
- Vertical vs horizontal: bigger server = simple but capped & single point of failure; scale-out handles growth but needs partitioning, coordination, consistency.
- Partitioning + replication: spread load/storage and enable parallelism; copies keep service available and recoverable.
- Cross-node challenges: all-commit-or-all-rollback coordination, network latency/failures (in-doubt txns), cross-node concurrency control, replica consistency.
- Why distributed SQL fits: SQL + integrity + ACID + scale + replication match every requirement. Evaluate first (any two): consistency needs, transaction latency, recovery, security, cost, operational complexity, migration effort.

### 1.9 Summary, homework, questions

- Summary: NewSQL = relational + SQL + scalability; ACID matters for reliable transactions; horizontal vs vertical scaling; partitioning/replication/concurrency control/recovery manage load & failure; NoSQL not uniform; NewSQL helps transactional apps but adds coordination/network/operational challenges; choose on workload, guarantees, scalability, availability, cost, operations.
- Homework H1–H4: (H1) ≥4 advantages & 3 limitations; (H2) compare SQL/NoSQL/NewSQL on models, interfaces, transactions, scalability → use table 1.5; (H3) how scale-out + partitioning + replication support a distributed transactional DB (capacity / parallelism / availability, with concurrency control and coordination keeping ACID, at the price of overhead); (H4) justify distributed SQL for the bank + two challenges (cross-node latency, network failures, complexity, cost, migration, security).
- Questions & memo covers: define NewSQL; ACID with transfer example; OLTP examples; vertical vs horizontal; non-linear scaling; partitioning vs replication; distributed transaction; five features; index trade-off; table comparison; 3 pros/3 cons; "always faster?" (no); match system↔feature; HTAP; Altibase caveat; scenarios (telecom → VoltDB, multi-region retailer → CockroachDB, bank factors).

> **Exam angle:** definition/central idea; ACID worked example; scaling & partition-vs-replication; the comparison table; pros vs cons; match systems to scenarios; banking case (40-mark style) — structure as *requirement → feature → trade-off*.

## 2 · Big Data technologies (1): Hadoop, Spark, cloud-native & governance `L21`

Agenda: what is Big Data (V5) · sources & characteristics · evolution of architectures · Hadoop legacy & lessons · Spark & cloud-native alternatives · HDFS vs cloud object stores · ethical & legal considerations · Spark practical (no slide detail — see practical sheet). Textbook: Ch 25 (only §25.1 examinable).

### 2.1 What is Big Data? — the V5

Data growth from social media, IoT, satellite imagery, app telemetry, eSIM, cloud logs, BI/user analytics. **Definition:** datasets whose size, speed, diversity or complexity need scalable storage/processing/management/analysis beyond traditional approaches. Suitability depends on data, workload, performance needs and infrastructure — no single best technology.

| Volume | Velocity | Variety | Veracity | Value |
| --- | --- | --- | --- | --- |
| TB → zettabytes (L13 said up to yottabytes — same point) | Real-time ingestion/processing | Structured, semi-, unstructured | Trustworthiness & quality | Business/analytical relevance |

Telecom illustrates all five (billions of records · fraud screening · calls/SMS/telemetry · duplicate/corrupt records · churn prediction).

- Generation points: mobile devices, microphones, readers/scanners, science facilities, software, social media, cameras, instant messaging.
- Application areas: homeland security, smarter healthcare, multi-channel sales, telecom, manufacturing, traffic control, trading, analytics, search, quality; later list — AI in healthcare, fraud detection, smart cities, IoT analytics, recommendation systems, climate modelling, genomics.

### 2.2 Evolution of architectures

- **2004 MapReduce** — Parallel batch processing on clusters
- **Hadoop v1/v2 + YARN** — Resource management + HDFS
- **2014+ Spark** — In-memory, DAG-based, faster
- **Now: cloud-native** — Serverless, autoscaling, object storage

Trend: **batch → real-time** and **on-premises → cloud**.

### 2.3 Hadoop — legacy components & lessons

- HDFS distributed file system · MapReduce batch programming model · YARN resource management (ResourceManager + NodeManager).
- MapReduce word count: map emits (word,1) → shuffle/sort groups by key → reduce sums. Scales via parallel maps on local data; intermediate results go to disk.
- Limitations: (1) intermediate results written to disk (overhead); (2) iterative algorithms repeatedly read/write; (3) batch, not low-latency/interactive; (4) these pushed adoption of Spark and stream-processing frameworks.

> **Note:** Hadoop is an **ecosystem** — don't blame HDFS/YARN for MapReduce's disk-write limitation.

### 2.4 Apache Spark

- Distributed engine for large-scale analytics: SQL, batch, structured streaming, ML, graph. Uses DAGs (stages as nodes, one-way dependencies, no cycles) to represent/schedule work.
- Can cache in memory when beneficial (also uses disk). Reads HDFS and object stores (S3, Azure Blob, Google Cloud Storage). Lower overhead for iterative/interactive jobs than MapReduce.

|  | Hadoop MapReduce | Spark |
| --- | --- | --- |
| Model | Map → shuffle → reduce | DAG of stages |
| Intermediate data | Commonly to disk | Cached in memory when useful |
| Best for | Batch | Batch, iterative, interactive, streaming, ML, graph, SQL |
| Status | Foundational, largely legacy | Modern, widely used |

### 2.5 Storage — HDFS vs cloud object storage

|  | HDFS | Cloud object storage |
| --- | --- | --- |
| Model | Distributed file system | Objects in buckets/containers |
| Infrastructure | Usually managed on a cluster | Managed by cloud provider |
| Scaling | Cluster capacity planning | Scales via the service |
| Access | File-system interfaces, cluster-based | API-based |
| Cost | Infrastructure + storage + operations | Usage-based; request & transfer charges |
| Common use | Hadoop processing | Data lakes, backups, analytics pipelines |

### 2.6 Ecosystem tools — then vs now

| Legacy | Modern | Distinction |
| --- | --- | --- |
| Pig | Spark SQL, dbt | Spark SQL processes distributed data; dbt manages SQL transformations |
| Hive | Trino, BigQuery | Hive = SQL warehousing on Hadoop; others have different distributed query architectures |
| Sqoop | NiFi, Airbyte | Sqoop moves RDBMS↔Hadoop; others do broader data integration |
| HBase | Cassandra, Cloud Bigtable | Wide-column NoSQL (L13) with different architectures/consistency models |

### 2.7 Flink · 2.8 Cloud platforms · 2.9 File formats · 2.10 Orchestration · 2.11 Streaming · 2.12 Query engines & ML

| Topic | Key points |
| --- | --- |
| **Apache Flink** | Distributed framework primarily for **stateful stream processing**: event-time processing, state management, windows; also batch. Fraud detection, IoT monitoring, real-time analytics (bank flags suspicious activity in seconds). Related: Kafka Streams, Spark Structured Streaming. *Stateful* = remembers across events; *event time* = when it happened; *window* = slice of stream. |
| **Cloud-native** | **AWS:** S3, Glue (integration/catalogue), EMR (managed processing), Kinesis (streaming), Redshift (warehouse). **Azure:** Data Lake Storage, Data Factory, Fabric/Synapse. **Google:** Cloud Storage, BigQuery, Dataflow, Pub/Sub. **Databricks:** lakehouse for engineering/analytics/ML. Managed services reduce the need to run every cluster component. |
| **File formats** | **Columnar** Parquet, ORC — read only needed columns (analytics). **Row-oriented** Avro — record exchange, schema-based serialisation. **Lake table formats** Delta Lake, Iceberg, Hudi — metadata, schema evolution, transactions. |
| **Orchestration** | Airflow, Prefect, Dagster — ETL pipelines: dependencies, scheduling, monitoring (retries, alerts). |
| **Streaming** | **Kafka** distributed event-streaming (stores/distributes events via topics); **Pulsar** alternative; **Kinesis / Pub/Sub** managed ingestion; **Flink / Spark Structured Streaming** compute on events. Supports replay, continuous processing, event-driven apps. Kafka = the "pipe", Flink = the "brain" (sensors → Kafka → Flink → alert). |
| **Query engines, analytics & ML** | Presto/Trino = distributed SQL engines; BigQuery, Snowflake, Databricks SQL = cloud warehouses (serverless, autoscaling, cost-efficient). ML at scale: Spark MLlib, TensorFlow, Vertex AI, SageMaker; unified training/deployment pipelines. |

### 2.13 Governance, security, ethics & legal

Protect data across its lifecycle: collection → storage → processing → sharing → deletion.

| Area | Requirement |
| --- | --- |
| Access control | RBAC or ABAC, least privilege |
| Encryption | In transit and at rest |
| Auditing & monitoring | Record access/admin activity |
| Data governance | Catalogues, ownership, lineage, retention, quality controls |
| Privacy & legal | South Africa's **POPIA**; EU **GDPR** where applicable |

**Discussion — risks of combining location + purchases + social media to predict behaviour:** privacy/re-identification & sensitive inferences · POPIA/GDPR (lawful basis, purpose limitation, consent, profiling) · bias/discrimination · security breach target · inaccurate inferences (veracity) · surveillance/manipulation.

### 2.14 Summary, homework, questions

- Summary: batch→real-time, on-prem→cloud; MapReduce/Hadoop foundational but largely legacy; Spark, cloud-native tools and ethical frameworks are essential; postgraduate readiness = hands-on, critical, ethical engagement.
- Homework — telecom case (millions of events/hour: store history; detect suspicious activity live; ML on history; protect subscriber data):
  - (a) history → cloud object storage data lake (Parquet/ORC + Delta/Iceberg); HDFS if on-prem Hadoop exists
  - (b) real-time → Flink or Spark Structured Streaming fed by Kafka/Kinesis/Pub-Sub
  - (c) Spark: Spark SQL batch analysis + MLlib training; DAG + in-memory caching help iteration
  - (d) controls: RBAC/ABAC least privilege; encryption in transit/at rest (also audit, governance, POPIA/GDPR, masking)
  - (e) no single tool: requirements differ (cheap storage / low-latency stateful streams / batch+ML / cross-cutting governance) → combine specialised components
- Questions & memo covers: define Big Data + V5; chronological order; Hadoop components; MapReduce limitations; ecosystem caveat; Spark + DAG; Spark not purely in-memory; HDFS vs object store; Pig/Hive/Sqoop/HBase matches; Flink; Kafka vs Flink; AWS/Google services; Parquet vs Avro; Delta/Iceberg/Hudi; Airflow; governance measures; scenarios (retailer risks; why Kafka/Flink and Spark).

> **Exam angle:** V5; evolution & why MapReduce's disk-based batch design motivated Spark; HDFS vs object storage; tool↔role matching; governance/POPIA/GDPR; telecom-style scenario (*requirement → technology → why → trade-off*). Also read textbook §25.1 (MapReduce/Hadoop).

## 3 · Data mining concepts `L22`

Agenda: data mining vs warehousing · KDD · goals · types of knowledge · association rules · classification · clustering · other algorithms (sequential, time series, regression, neural networks, genetic algorithms) · applications · tools & future. Textbook: Ch 28.

### 3.1 Definition, mining vs warehousing, KDD, goals

> **Data mining** = analysing large datasets to discover useful patterns, relationships, trends and predictive models, using statistics, ML, database systems and pattern recognition to support decisions (items bought together, fraud prediction, customer grouping, demand forecasting).

|  | Data warehousing | Data mining |
| --- | --- | --- |
| Purpose | Integrate & organise data for reporting/analysis | Discover patterns, build predictive/descriptive models |
| Activities | Integration, storage, querying, reporting | Association rules, classification, clustering, regression |
| Output | Curated datasets, reports, views | Patterns, rules, clusters, models |
| Relation | Source of historical integrated data | Can analyse warehouse *or* other sources |

> **Note:** A warehouse is **not a prerequisite** — mining works on operational DBs, data lakes, other datasets if accessible/preparable.

**KDD — broader, iterative; data mining is one stage**

① Data selection → ② cleansing (missing/inconsistent/duplicate/incorrect) → ③ enrichment (add data/background knowledge) → ④ transformation (suitable representation) → ⑤ **data mining** (algorithms) → ⑥ evaluation & presentation. May loop back (e.g. supermarket: select 2025 tills → clean → add category/region → baskets → mine → show strong rules).

**Goals — PICO (goals, not techniques)**

| Prediction | Identification | Classification | Optimisation |
| --- | --- | --- | --- |
| Estimate unknown/future outcome (next month's electricity demand) | Detect presence of item/event (fraudulent transactions) | Assign to predefined categories (spam vs legit) | Best use of limited resources under constraints (delivery vehicle allocation) |

Classification, regression, clustering are *techniques*; "classification" appears as both goal and technique. **Types of discovered knowledge:** association rules · classification hierarchies · sequential patterns · patterns within time series · clustering.

### 3.2 Association rules

- From market-basket data (items bought in one visit = itemset). Rule X ⇒ Y (distinct item sets); interesting only if it meets minimum support and minimum confidence.
- Support = % of transactions containing all items of LHS ∪ RHS. Confidence = P(RHS | LHS) = support(LHS ∪ RHS) / support(LHS).

**Data:** 101 {milk, bread, cookies, juice}; 792 {milk, juice}; 1130 {milk, eggs}; 1735 {bread, cookies, coffee}.

| Rule | Support | Confidence |
| --- | --- | --- |
| milk ⇒ juice | 2/4 = 50% | 2/3 = 66.7% |
| bread ⇒ juice | 1/4 = 25% | 1/2 = 50% |

Support uses the whole DB as denominator; confidence uses LHS transactions. **Confidence is not symmetric** (juice ⇒ milk = 100%).

- Two steps: (1) find all large/frequent itemsets above support threshold; (2) for itemset X and subset Y, Z = X − Y: rule Z ⇒ Y valid if support(X)/support(Z) > min confidence.
- Search reduction: downward closure — every subset of a frequent itemset is frequent; anti-monotonicity — every superset of an infrequent itemset is infrequent (prune it).

### 3.3 Algorithms for frequent itemsets

> **Apriori** (Agrawal & Srikant, 1994) — level-wise: scan for 1-itemset supports → L₁ (≥ mins) → candidates C_(k+1) by joining L_(k) members sharing k−1 items → prune any with an infrequent size-k subset → scan to count → L_(k+1); stop when empty.
>
> **Trace (mins 50%):** L₁ = milk(75%), bread, cookies, juice (eggs, coffee 25% dropped). L₂ = {milk,juice}, {bread,cookies} (50% each). Members share no item → C₃ = ∅ → stop. Rules at 70% confidence: juice ⇒ milk 100% ✓, bread ⇒ cookies 100% ✓, cookies ⇒ bread 100% ✓, milk ⇒ juice 66.7% ✗.

| Algorithm | Main idea | DB scans |
| --- | --- | --- |
| **Apriori** | Level-wise candidates + downward-closure pruning | One per level |
| **Sampling** | Mine in-memory samples; check superset property by scanning rest. **Negative border** = minimal itemsets not frequent but all subsets frequent (closest possible extra frequent sets); if one proves frequent → second pass needed | 1–2 |
| **FP-tree / FP-growth** | Compress DB into tree; no candidate generation. Build: frequent 1-itemsets sorted by count desc, root "null", insert each transaction's frequent items in order (shared prefix → increment count, else new node). FP-growth: single path → combos (support = min node count); else per header item build conditional pattern base/FP-tree and recurse | 2 |
| **Partition** | Split into non-overlapping partitions → local frequent itemsets → union as global candidates → scan whole DB for global support. A globally frequent itemset is frequent in ≥1 partition | 2 |

```
FP-tree (min support 2):  null
 ├─ milk(3) ─┬─ bread(1) ─ cookies(1) ─ juice(1)   ← 101
 │           └─ juice(1)                            ← 792
 └─ bread(1) ─ cookies(1)                           ← 1735
```

### 3.4 Hierarchies, multi-dimensional & negative rules, complications

- Hierarchies (taxonomies): items grouped in classes/subclasses (supermarket: beverages [carbonated, non-carbonated: juices, water, wine coolers], desserts [ice cream, baked, frozen yogurt]). "beverages ⇒ desserts" may fail while lower-level rules (healthy frozen yogurt ⇒ bottled water; rich ice cream ⇒ wine cooler) succeed.
- Multi-dimensional: several attributes, e.g. Time(6:30…8:00) ⇒ Items_bought(milk) (support 50%, confidence 100% on the data). Quantitative attributes → non-overlapping intervals (salary bands ⇒ car brands).
- Negative rules: e.g. buyers of "Wakeup" drink unlikely to buy "Nightos" chips — hard because support is very low by definition.
- Complications: huge itemset cardinality · variation by location/season · multi-dimensional item classification · variable data quality (missing, erroneous, conflicting, redundant).

### 3.5 Classification

- Supervised: predetermined classes, learned from a pre-classified training set; model = decision tree or rule set.
- Build_tree: all same class → leaf; no attributes left → majority-class leaf; else split on attribute with highest information gain (best separates classes); a branch per value; empty subset → leaf with parent's majority class; recurse on remaining attributes.

```
Salary
├─ < 20K       → no
├─ 20K–50K     → Age ─ < 25 → no ; ≥ 25 → yes
└─ ≥ 50K      → yes
```

Each root-to-leaf path = one IF…THEN rule (e.g. 20K ≤ salary < 50K AND age ≥ 25 → yes). Salary 35K/age 30 → yes; salary 15K → no.

### 3.6 Clustering

- Unsupervised: groups records by similarity (distance measure), no predefined labels. Uses: customer segments, network device traffic, anomalies in security logs.
- K-means (input m records, k): init k centroids (e.g. k-means++) → assign each record to nearest centroid → recompute centroids as means → repeat to convergence. Minimises within-cluster sum of squared distances. Depends on initialisation, k, feature scaling, outliers; suits numeric data, weak on clusters of varied size/density/non-spherical shape.
- Trace: values 1,2,10,11, k=2, init 1 & 10 → {1,2} / {10,11}; centroids 1.5 and 10.5; no change → converged.

|  | Classification | Clustering |
| --- | --- | --- |
| Learning | Supervised | Unsupervised |
| Labels | Predefined | None — groups discovered |
| Output | Tree / rules | Clusters + centroids |

### 3.7 Other data mining approaches

| Approach | Key points | Typical goal |
| --- | --- | --- |
| **Sequential patterns** | Time-ordered transactions form sequences of itemsets; support of S = % of sequences containing S as a subsequence (phone → case → screen protector) | Prediction |
| **Time series** | Observations over time (stock prices, hourly electricity, traffic, sales) — trends, seasonality, anomalies, forecasting. Usually numeric; sequential patterns = ordered event sequences; can overlap | Prediction, trends |
| **Regression** | Special application of classification rule: function mapping variables to a target; Y = f(x₁…xₙ); linear f → **linear regression**. Slide: LAB_TESTS → P (probability of survival) | Numeric prediction |
| **Neural networks** | Layered, weighted units; supervised or unsupervised/self-supervised; images, text, anomaly, forecasting, classification (time series: recurrent, temporal conv., Transformers). Model complex relations but need data/compute and are hard to interpret | Prediction, classification, identification |
| **Genetic algorithms** | Randomised evolution-based search: population → fitness → select → crossover → mutation → new population → repeat. Delivery-route example. Heuristic: no global-optimum guarantee | Optimisation |
| Association rules / classification / clustering | Unsupervised discovery · supervised · unsupervised segmentation | Co-occurrence · classification/identification · segmentation/anomalies |

### 3.8 Applications, tools & future

- Applications: Marketing (consumer behaviour, advertising, service location, segmentation, targeted mailing) · Finance (fraud, creditworthiness, investment performance) · Manufacturing (optimise men/machines/materials) · Health (image & genetic analysis, drug side effects, treatment effectiveness, cost optimisation).
- Commercial tool techniques: association rules, clustering, neural networks, sequencing, statistics, decision trees, regressions; advanced: GAs, case-based reasoning, Bayesian networks, nonlinear regression, combinatorial optimisation, pattern matching, fuzzy logic.
- Tools: Python (pandas, scikit-learn) · R · Spark MLlib (distributed) · BigQuery ML (model near data) · KNIME (visual workflows). Choose by dataset size, compute, requirements, cost, explainability, integration.
- Future: AI/ML/statistics/optimisation algorithms; mining TB/PB in big-data systems (Mahout on Hadoop, Spark MLlib — L21); data in cloud warehouses accessed through OLAP servers (L23–24); business analytics is Spark's primary goal.

**Questions & memo covers:** definition; warehouse vs mining; KDD stages; PICO; support/confidence calculations (both directions); two-step rule generation; downward closure; Apriori trace; sampling/negative border; FP-tree trace; partition; hierarchy and multi-dimensional rules; negative rules; complications; decision-tree tracing; clustering vs classification; k-means trace; sequential vs time series; regression; NN pros/cons; GA steps; technique-selection scenario; tools; link to big data.

> **Exam angle:** KDD + PICO; support/confidence table calculations; tracing Apriori; algorithm descriptions; reading a decision tree into rules; a k-means iteration; picking the right technique for a scenario (name it, justify vs data/goal, note limitations).

## 4 · Data warehousing & OLAP `L23` `L24`

`L23` agenda: BI & data warehousing · business environment · operational vs informational systems · BI · warehouse architecture & components · data mart and reasons for one · plus (slides) dimensional modelling, star schema, information packages, cube, hierarchies, STARjoin/STARindex/materialised views, SQL.<br>`L24` agenda: continuation · implementing a ROLAP schema · OLAP extensions to SQL · how Oracle supports OLAP · plus slowly changing dimensions, MERGE loading, windowing/ranking, BI tools, OLAP categories, homework. Textbook: Ch 29.

### 4.1 Concepts & architecture

**Business environment `L23`**

Purpose: make money · sell products/services · to customers · sourced from suppliers/internal · organised by business units/processes · decisions by managers · threats: losing customers, closing down → managers need timely, accurate information (informational systems).

**Operational vs informational systems `L23`**

|  | Operational (OLTP) | Informational (OLAP) |
| --- | --- | --- |
| Metaphor | "Make the wheels turn" | "Watch the wheels turn" |
| Purpose | Record/process transactions as they happen | Support informed decisions |
| Users | Shop-floor workers | Managers, analysts |
| Examples | POS, inventory, general ledger, claims | "Top-selling products", "problem branches", alerts when below target |
| Data | Current, detailed, application-specific | Current + historical, integrated, analysis-oriented |

Running big reports on the live till DB would slow the tills → separate warehouse. Link L20: OLTP = NewSQL's workload; OLAP = analytical counterpart.

**Dashboards** (day-to-day management): like instruments when driving a car/flying an aircraft — key indicators, trends, comparisons at a glance (trend line of net promoter score, gauges per product line).

**Business intelligence `L23`**

Applications & technologies to **gather, access and analyse** data about business operations → in-depth knowledge of customer/supplier behaviour, sales & production volumes, internal operations → better manager decisions.

**Why a warehouse is viable for informational systems; definition `L23`**

(1) data from multiple applications, (2) current + historical content, (3) designed for analysis (key measures & dimensions).

> **Data warehouse** = collection of data supporting management decisions that is **subject-oriented** (sales, policies, claims) · **integrated** (from different operational systems) · **non-volatile** (read-oriented; updated only via controlled loading/correction) · **time-variant** (time stored explicitly → day/week/month/quarter/year analysis).

**Architecture, ETL/ELT & roles `L23`**

| # | Component | # | Component |
| --- | --- | --- | --- |
| 1 | Source data (internal systems; external: suppliers, partners, government) | 6 | Information delivery (data mining, OLAP, reports/queries) |
| 2 | Data staging — ETL / ELT | 7 | Interactive users |
| 3 | Enterprise data warehouse (maybe cloud) | 8 | Management & control |
| 4 | Metadata (data about data) | 9 | Hardware, OS, DBMS |
| 5 | Data mart | 10 | DB personnel |

- ETL: extract → transform in staging (clean, standardise e.g. M/F vs 1/2, one date format, dedupe) → load. ELT: extract → load raw → transform inside the (cloud) platform (L21).
- Roles: data transformation specialists (extraction/integration/transformation) · warehouse administrators (what data, when, how) · data modellers (relational + dimensional) · business analysts (design/create reports).

**The data mart `L23`**

Smaller, focused subset of the warehouse for one department/subject (Sales mart, Finance mart); the star schema lives in the data mart. *Reasons (not on slides — check textbook):* department focus · faster queries (less data) · quicker/cheaper/lower-risk build · easier for users · access control · departmental ownership.

### 4.2 Dimensional modelling & the star schema

**Dimensions, questions, information packages `L23`**

| Business | Subject | Dimensions |
| --- | --- | --- |
| Supermarket | Sales | Time, branch, product, promotion |
| Airline | Frequent flyers | Time, customer, flight, fare class, airport |
| Hotel chain | Hotel occupancy | Time, hotel branch, room type |

Time is very common (historical analysis) but **not compulsory**. Example question — *"revenue of luxury rooms month by month in Gauteng by hotel branch"*: measure = revenue; room type = luxury; time = month; hotel = province Gauteng, level branch → need structure supporting dimensional analysis = star schema.

**Information package** (one per subject, built while gathering requirements): columns = dimensions with hierarchy levels top → bottom; plus facts. Hotel occupancy: **Time** Year/Quarter/Month · **Hotel** Hotel line/Province/Branch · **Room type** Room type/Size · **Facts** occupied rooms, vacant rooms, revenue. It maps to dimension tables (columns) + fact table (facts).

**Star schema = ROLAP `L23`**

- Central fact table (business event at defined granularity: foreign keys + measures) surrounded by dimension tables; FKs in fact table = one-to-many links. Dimensional view using relational structures and queries (Relational OLAP).
- Hierarchy levels are denormalised into one wide dimension table (fewer joins).

```
OCCUPANCY_FACTS(Time_Key FK, Hotel_Key FK, Room_Key FK, Occupied_rooms, Vacant_rooms, Revenue)
HOTEL(Hotel_Key PK, Hotel_code, Hotel_Line, Province, Branch)
RM_TYPE(Room_Key PK, Room_code, Room_Type, Size)
TIME(Time_Key PK, Time_code, Year, Quarter, Month)
```

```
SELECT h.Province, t.Year, t.Month, SUM(f.Revenue)
FROM OCCUPANCY_FACTS f
JOIN HOTEL h ON f.Hotel_Key=h.Hotel_Key
JOIN RM_TYPE r ON f.Room_Key=r.Room_Key
JOIN TIME t ON f.Time_Key=t.Time_Key
WHERE r.Room_Type='Luxury'
GROUP BY h.Province, t.Year, t.Month;
```

Pattern: fact table in `FROM`, **star join** to each dimension, `WHERE` filters on dimension attributes, `GROUP BY` dimension levels, `SUM` the measure.

**Data cube, roll-up & drill-down, performance `L23`**

- Data cube (hypercube): one axis per dimension, each cell one measure value (Feb × Luxury × Gauteng = R20m). Conceptual — not necessarily stored as a cube (ROLAP: tables). 3 months × 2 types × 2 provinces = 12 cells.
- Roll-up = up the hierarchy, more summarised (months → quarter; branch → province → hotel line). Drill-down = down, more detail (quarter → months: Jan 5m + Feb 10m + Mar 5m = Qtr1 20m). The two R20m figures on the slides are different illustrations.
- STARjoin: joins fact table with multiple dimension tables. STARindexes: specialised indexes; bitmap indexes for low-cardinality attributes (e.g. Room_Type). Materialised views: precomputed stored query results; must be refreshed when data changes (speed vs storage/maintenance).

### 4.3 Building & loading the ROLAP structure `L24`

**Recap `L24`**

- Dimension tables + fact table for a data mart; a dimension may have several hierarchies (TIME: calendar_rollup, fiscal_rollup).
- Surrogate keys = system-generated (Time_Key, Hotel_Key, Room_Key); distinct from business keys (Hotel_ID, Room_ID, Time_ID). They let a dimension hold multiple historical versions of the same entity (same Hotel_ID, new Hotel_Key).

**SQL for warehousing `L23` `L24`**

| Task | Statements |
| --- | --- |
| Create ROLAP structure | `CREATE SEQUENCE` (surrogate keys) · `CREATE TABLE` · `CREATE DIMENSION` (Oracle-only) · `CREATE MATERIALIZED VIEW` (speed) |
| Load data | `MERGE` · `UPDATE/INSERT` (initial + incremental loads) |
| Reports | `SELECT … GROUP BY CUBE \| ROLLUP \| GROUPING SETS` |

```
CREATE DIMENSION HOTELS_DIM
  LEVEL Hotel IS HOTEL.Hotel_ID   LEVEL Province IS HOTEL.Province_ID   LEVEL Hotelline IS HOTEL.Hotelline_ID
  HIERARCHY Hotel_Rollup (Hotel CHILD OF Province CHILD OF Hotelline)
  ATTRIBUTE Hotel DETERMINES (Hotel_name, Hotel_address, Hotel_city, ...)
  ATTRIBUTE Province DETERMINES Province_name   ATTRIBUTE Hotelline DETERMINES Hotelline_desc;
```

`LEVEL … IS` = level + identifying column; `HIERARCHY … CHILD OF` = roll-up path lowest → highest; `ATTRIBUTE … DETERMINES` = functional dependency. Declares metadata only (helps query rewrite/materialised views); Oracle-specific.

**Slowly changing dimensions (SCD) `L24`**

Warehouses keep long history while dimension data (e.g. marital status) changes → need a strategy.

| Type | Rule | History |
| --- | --- | --- |
| **1** | Overwrite the changed value | Lost (corrections only) |
| **2** | New record per change | Full history (one row per version) |
| **3** | Alternate column holds old value in same record | Limited (previous + current) |

**Type 2 example** CUSTOMER(Cust_Key, Cust_ID, Eff_start, Eff_end, Cust_sname, Marital_stat): Cust_Key 12345, ID 850002, 01/03/2015–09/03/2018, single, Jones → new row Cust_Key 12347, same ID, from 10/03/2018, end blank (current), married, Charming. Old facts keep pointing to 12345, new ones to 12347. (Type 1: one row "married" — past sales look married. Type 3: extra Prev_marital_stat column.)

**Loading with MERGE `L24` `L23`**

```
MERGE INTO CUSTOMER C
USING (SELECT Cust_ID, ... FROM CUST_STG) S
ON (C.Cust_ID = S.Cust_ID AND C.Marital_Status = S.Marital_stat)
WHEN MATCHED     THEN UPDATE SET ...        -- type 1 change
WHEN NOT MATCHED THEN INSERT VALUES (...);  -- type 2 change or new customer
```

- Initial load (empty dimension → everything NOT MATCHED → inserted) vs incremental loads (updates + inserts). MERGE = "update if exists else insert" in one statement (same idea as Cypher MERGE, L18).
- Changed marital status ⇒ no match (ON includes status) ⇒ INSERT = Type 2.
- Caution: a full Type 2 load also needs to set End_date on the old row and take a new surrogate key from the sequence (extra UPDATE step).

### 4.4 OLAP extensions to SQL `L24`

- Why: plain SQL can't easily do % change vs a year ago, moving averages, cumulative sums, statistics. ANSI added OLAP functions (SQL:1999 amendment → SQL:2003; slides also say "since SQL 2008" for CUBE/ROLLUP/GROUPING SETS — dates inconsistent; point is they're standard and widely supported).
- Capabilities: extended grouping (ROLLUP, CUBE, GROUPING SETS) · windowing (cumulative/moving SUM, AVG, MIN, COUNT over row ranges) · ranking (RANK, DENSE_RANK, cumulative distribution, percent rank, N-tiles: quartiles/deciles/percentiles).

**Extended grouping**

Traditional query `GROUP BY province, month` gives one grouping only, no subtotals (would need several queries + UNION).

| Operation | Purpose | Groups for (province, month) |
| --- | --- | --- |
| **CUBE** | All combinations | (p,m), (p), (m), grand total — 2² = 4 |
| **ROLLUP** | Hierarchical subtotals + grand total, right to left | (p,m), (p), grand total |
| **GROUPING SETS** | Only listed groupings; `()` = grand total | e.g. ((p,m),(p),()) = same as ROLLUP |

**Example** (Gauteng Jan 10, Feb 20; KZN Jan 5, Feb 15): ROLLUP = 7 rows (4 detail + Gauteng 30 + KZN 20 + grand 50); CUBE = 9 rows (adds month totals Jan 15, Feb 35). `NULL` = summarised level; `GROUPING(col)` = 1 for subtotal NULLs, 0 for real data.

**Windowing**

- 3-month moving average of 1,4,1,2,3,4 → Mar 2, Apr 7/3 ≈ 2.33, May 2, Jun 3 (first two months lack full history on the slide table).
- SQL: AVG(SUM(saleAmount)) OVER (ORDER BY yearMonth ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) — inner SUM + GROUP BY gives monthly totals; OVER sorts and defines frame = current + 2 previous rows. Results for B003: avg 21,000 / 28,000 / 32,000 / 39,000 / 42,000 / 43,000; sums 21,000 / 56,000 / 96,000 / 117,000 / 126,000 / 129,000 (SQL uses available rows for Jan/Feb).

**Ranking**

`RANK() OVER (ORDER BY SUM(saleAmount) DESC)` / `DENSE_RANK()`. Pretoria branches: B009 120m (1,1); B018/B022/B028 92m (2,2); B033 45m (**RANK 5**, **DENSE 3**); B046 42m (6, 4). RANK leaves gaps after ties; DENSE_RANK doesn't. (Qualify `Branch.branchNo` to avoid ambiguity.)

### 4.5 Tools, OLAP categories & homework `L24`

> **Modern DW & BI tools**
>
> - Data warehousing: Oracle Autonomous Data Warehouse
> - Integration: Oracle Data Integration
> - Development: Oracle SQL Developer
> - BI: Power BI, Tableau, Oracle Analytics Cloud
> - Open-source BI: Apache Superset
>
> Compare L21 cloud platforms (BigQuery, Redshift, Snowflake, Databricks).

> **OLAP tool categories**
>
> - MOLAP — multidimensional (cube) storage, fast, precomputed
> - ROLAP — dimensional view on relational tables (this module)
> - HOLAP — hybrid (detail relational, summaries cube)
> - DOLAP — desktop OLAP, local cubes
>
> Category descriptions added; slide gives names only.

**Homework**

- (1) Open-source & BI: research PostgreSQL or MySQL and OLAP. Pointers: PostgreSQL has ROLLUP/CUBE/GROUPING SETS, window functions, materialised views; MySQL 8+ has window functions and WITH ROLLUP but lacks CUBE/GROUPING SETS and native materialised views; neither has CREATE DIMENSION; Superset as front-end. Verify against official docs.
- (2) SQL tasks (hotel schema): ① TIME table + CREATE DIMENSION with hierarchy DAY → MONTH → QUARTER → YEAR (slide text missing an arrow) · ② revenue per hotel per month with GROUP BY CUBE (Hotel_ID, Month_ID) → detail, hotel totals, month totals, grand total · ③ per hotel per quarter with ROLLUP (Hotel_ID, Quarter_ID) → detail, hotel subtotal, grand total (column order matters) · ④ GROUPING SETS ((Hotel,Month),(Room_Type,Month),(Hotel,Room_Type,Month)) → only those three groupings, NULL = column absent.

**Questions & memo covers:** `L23` operational vs informational; BI; why warehouse; four characteristics; components; ETL vs ELT; roles; dimensions per business; time dimension; question analysis; information package; data mart; star schema design + SQL; cube; roll-up/drill-down tracing; STARjoin/index/materialised views; CREATE DIMENSION; ROLLUP vs CUBE groupings; scenario (slow tills → warehouse/mart). `L24` surrogate keys; ROLAP statements; CREATE DIMENSION clauses; why SCD; Types 1/2/3; Type 2 tracing; initial vs incremental load; MERGE tracing; plain-SQL gaps; CUBE/ROLLUP/GROUPING SETS rows (9 vs 7); `()`; moving average; window syntax; RANK vs DENSE_RANK; N-tiles/BI tools; OLAP categories; homework-style code; scenario (ROLLUP + RANK).

> **Exam angle:** OLTP vs OLAP; four warehouse characteristics; architecture/ETL vs ELT/roles; info package → star schema + star-join SQL; roll-up vs drill-down; STARjoin/bitmap/materialised views; surrogate keys; SCD 1/2/3 before-after tables; MERGE logic; predicting rows/NULLs of CUBE/ROLLUP/GROUPING SETS; moving averages; RANK vs DENSE_RANK; MOLAP/ROLAP/HOLAP/DOLAP. Scenario: identify subject, dimensions, hierarchies, measures → design schema.

## 5 · Connections & exam focus

### Cross-lecture links

| From | To | Link |
| --- | --- | --- |
| `L20` OLTP | `L23` operational systems | Same OLTP workload; OLAP is its analytical counterpart. TiDB's HTAP bridges both. |
| `L20` NoSQL comparison | L13–19 (MongoDB, Neo4j) | NoSQL varies in model/query/transactions; CAP trade-off explains NewSQL's strong-consistency choices. |
| `L20` Kafka integration (VoltDB) | `L21` streaming | Kafka = event pipe feeding real-time transaction/stream engines. |
| `L21` big data / V5 / HBase, Cassandra | L13 big data & NoSQL | Wide-column stores; L21 adds the processing side. |
| `L21` Spark MLlib, Mahout | `L22` tools & future | Mining at TB/PB scale uses big-data libraries; business analytics is Spark's goal. |
| `L21` ELT, cloud warehouses (BigQuery, Redshift) | `L23` ETL/ELT; `L24` modern tools | Cloud warehouses load raw then transform in-platform. |
| `L22` data mining | `L23` warehouse | Mining can use a warehouse (but doesn't need one); warehouse feeds "information delivery" (data mining, OLAP, reports); OLAP servers bring cloud-warehouse data to mining. |
| `L23` CREATE DIMENSION, MERGE, GROUP BY ROLLUP/CUBE | `L24` | L23 introduces the statements; L24 explains them in depth, adds SCD, windowing, ranking. L18's Cypher MERGE = same match-or-create idea. |
| `L23` star schema, hierarchies | `L24` surrogate keys, hierarchies | Same hotel schema; surrogate keys enable SCD Type 2 versions. |

### Discrepancies to remember

- L21 volume "TB→zettabytes" vs L13 "up to yottabytes" — same point (enormous scale).
- L22: "classification" is both a PICO goal and a mining technique.
- L24: SQL OLAP standardisation dates inconsistent (SQL:1999/2003 vs "since SQL 2008"); L24 MERGE slide is a simplified Type 2 pattern; homework hierarchy typo (missing arrow).
- L23: data-mart reasons aren't on the slides (read textbook).

### Scenario cheat-sheet

| Scenario cue | Reach for |
| --- | --- |
| Bank/e-commerce needing SQL + ACID + scale-out | Distributed SQL / NewSQL (CockroachDB, TiDB, NuoDB) |
| Telecom events, very high throughput, billing | VoltDB (or Altibase if history on disk too) |
| Real-time fraud/IoT alerting | Kafka + Flink / Spark Structured Streaming |
| Long-term cheap history + ML | Cloud object store data lake (Parquet + Delta/Iceberg) + Spark SQL/MLlib |
| Subscriber data protection | RBAC/ABAC least privilege, encryption, audit, POPIA/GDPR |
| Products bought together | Association rules (Apriori / FP-growth) |
| Good/bad risk from labelled history | Classification (decision tree) |
| Customer segments, anomalies | Clustering (k-means) |
| Delivery routing under constraints | Genetic algorithm (optimisation) |
| Forecast hourly demand | Time series / regression / neural networks |
| Heavy analysis slows the tills | Warehouse/data mart via ETL/ELT, star schema |
| Dimension attribute changes, need history | SCD Type 2 (surrogate key + effective dates) |
| Subtotals + grand total / all combos / chosen groupings | ROLLUP / CUBE / GROUPING SETS |
| Moving average, ranking with ties | Window functions; RANK vs DENSE_RANK |

## Heading coverage map

Every top-level heading from the five note files and where it appears above.

| Lecture | Source headings | Section |
| --- | --- | --- |
| `L20` | 1 Overview · 2 Introduction · 3 ACID & OLTP · 4 Why NewSQL / scaling · 5 Key features (+partitioning vs replication) · 6 SQL vs NoSQL vs NewSQL · 7 Advantages & limitations · 8 Examples (VoltDB, CockroachDB, NuoDB, TiDB, Altibase) · 9 Banking case study · 10 Summary · 11 Homework · 12 Questions & Memo | 1.1–1.9 |
| `L21` | 1 Overview · 2 What is Big Data (V5, generation points, applications) · 3 Evolution · 4 Hadoop · 5 Spark · 6 HDFS vs object storage · 7 Ecosystem tools · 8 Flink · 9 Cloud-native · 10 File formats · 11 Orchestration · 12 Streaming · 13 Query engines/analytics/ML · 14 Governance & ethics · 15 Applications · 16 Summary · 17 Homework · 18 Questions & Memo | 2.1–2.14 |
| `L22` | 1 Overview · 2 Definition, vs warehousing, KDD, PICO, types of knowledge · 3 Association rules · 4 Frequent-itemset algorithms (Apriori, Sampling, FP-tree, Partition) · 5 Hierarchies, multi-dimensional, negative rules, complications · 6 Classification · 7 Clustering (k-means) · 8 Other approaches (sequential, time series, regression, NN, GA) · 9 Applications · 10 Tools & future · 11 Questions & Memo | 3.1–3.8 |
| `L23` | 1 Overview · 2 Business environment · 3 Operational vs informational (+dashboards) · 4 BI · 5 Why warehousing viable · 6 Definition · 7 Architecture (+ETL/ELT) · 8 Roles · 9 Dimensional modelling (dimensions, questions, information packages) · 10 Data mart · 11 Star schema (+SQL) · 12 Data cube · 13 Roll-up/drill-down · 14 STARjoin/STARindex/materialised views · 15 SQL for warehousing (CREATE DIMENSION, MERGE, ROLLUP/CUBE/GROUPING SETS) · 16 Questions & Memo | 4.1–4.3 (+4.5) |
| `L24` | 1 Overview · 2 Recap ROLAP · 3 SQL statements (SEQUENCE, DIMENSION) · 4 Slowly changing dimensions · 5 Loading & MERGE · 6 Why SQL extensions · 7 CUBE/ROLLUP/GROUPING SETS · 8 Windowing & ranking · 9 Modern tools · 10 OLAP categories · 11 Homework · 12 Questions & Memo | 4.3–4.5 |
