# System Design + ML System Design Cheat Sheet

## 1. Core System Design Concepts

| Term | What it means | When to use it |
|---|---|---|
| **Horizontal scaling** | Add more machines | Default for stateless services at scale |
| **Vertical scaling** | Bigger machine | Quick fix, limited ceiling, SPOF risk |
| **Load balancer** | Distributes requests across servers (Round Robin, Least Connections, Consistent Hashing) | Any multi-instance service |
| **Consistent hashing** | Maps keys to nodes so minimal redistribution happens when nodes change | Sharding, routing same-domain/client traffic to same worker |
| **CAP theorem** | Can't have Consistency + Availability during a network Partition | Frame any distributed-storage tradeoff discussion |
| **Strong consistency** | Every read sees latest write | Financial transactions, account balances |
| **Eventual consistency** | Replicas converge over time | High-read systems tolerant of slight staleness (caches, crawler metadata) |
| **Sharding/Partitioning** | Split data across nodes by a key (e.g., client_id, hash(domain)) | Scale writes/reads horizontally |
| **Replication** | Copies of data on multiple nodes | Fault tolerance, read scaling |

## 2. Caching

| Term | Meaning |
|---|---|
| **Cache-aside** | App checks cache first; on miss, reads from DB and populates cache |
| **Write-through** | Write goes to cache and DB simultaneously |
| **Write-back** | Write goes to cache first, flushed to DB later (faster, riskier) |
| **TTL** | Time-to-live; auto-expire stale cache entries |
| **LRU/LFU eviction** | Evict Least Recently/Frequently Used when cache is full |

## 3. Rate Limiting & Pagination

| Term | Meaning | Use case |
|---|---|---|
| **Token bucket** | Bucket refills at fixed rate, allows bursts up to bucket size | Per-domain or per-client API limits |
| **Leaky bucket** | Fixed drain rate, no bursts | Smoothing output traffic |
| **Fixed window counter** | Count per fixed time block | Simple, but bursty at window edges |
| **Sliding window** | Smooths the fixed-window edge problem | More accurate rate limiting |
| **Offset pagination** | `LIMIT x OFFSET y` | Simple, static data only — breaks on live data |
| **Cursor/keyset pagination** | Opaque token marking last-seen item | Live/changing datasets — stable, indexed, fast |

## 4. Crawling / Large-Scale Data Pipelines

| Term | Meaning |
|---|---|
| **URL Frontier** | Priority queue of URLs to crawl, ordered by importance/staleness/change-frequency |
| **Bloom filter** | Probabilistic set membership check; false positives possible, no false negatives | Cheap URL dedup before fetch |
| **Content fingerprint (SimHash/MinHash)** | Detects near-duplicate content (not just exact-match) | Dedup across multiple sources reporting same story |
| **PageRank** | Importance score from inbound link structure; computed iteratively over the whole graph | Crawl prioritization (run as periodic batch job, not per-request) |
| **CDC (Change Data Capture)** | Stream of changes from a source DB | Real-time pipeline triggers |
| **Circuit breaker** | Stop sending requests to a failing/slow dependency for a cooldown period | Protect fleet from malicious/slow sites |
| **Fetcher vs Parser separation** | Fetch = I/O-bound; Parse = CPU-bound | Scale independently |

## 5. Database Types — Quick Picker

| Type | Examples | Best for |
|---|---|---|
| **Relational (SQL)** | Postgres, MySQL | ACID transactions, structured relational data (billing) |
| **Key-Value** | Redis, DynamoDB | Fast single-key lookups, caching, rate-limit counters |
| **Wide-column** | Cassandra, Bigtable | Massive write throughput, simple key-based access |
| **Document** | MongoDB, Elasticsearch | Flexible/semi-structured JSON records |
| **Search index** | Elasticsearch, Solr | Full-text search, fuzzy/relevance ranking |
| **Object storage** | S3, GCS | Large unstructured blobs (raw HTML, files) |
| **Graph** | Neo4j, Neptune | Relationship traversal (link graphs, fraud rings) |
| **Time-series** | InfluxDB, TimescaleDB | Timestamped metrics, range queries |

## 6. ACID vs BASE

| ACID (SQL) | BASE (NoSQL, distributed) |
|---|---|
| Atomicity, Consistency, Isolation, Durability | Basically Available, Soft state, Eventual consistency |
| Strong guarantees, harder to scale | Loose guarantees, scales horizontally |

## 7. ML System Design Fundamentals

| Term | Meaning |
|---|---|
| **Problem framing** | Translate business need → classification / regression / ranking / retrieval task |
| **Feature engineering** | Tabular features (counts, ratios) vs. learned embeddings (dense vectors) |
| **Embedding** | Dense vector representation capturing semantic meaning (used for similarity search) |
| **NER (Named Entity Recognition)** | Tag spans of text as entities (PERSON, ORG, DATE, MONEY) |
| **BIO tagging** | Begin/Inside/Outside scheme for multi-word entity spans |
| **Entity linking/disambiguation** | Map an extracted mention to a canonical entity ID (e.g., "Apple" → Apple Inc.) |
| **Batch inference** | Precompute predictions offline, serve from lookup — for non-urgent freshness |
| **Online inference** | Real-time prediction per request — low latency required |
| **Model drift / data drift** | Real-world data distribution shifts, degrading model accuracy over time |
| **A/B testing / shadow traffic** | Test new model against current one before full rollout |
| **Accuracy vs latency vs cost** | The ML equivalent of CAP theorem — always a three-way tradeoff |

## 8. Failure Mode / Resilience Vocabulary

| Term | Meaning |
|---|---|
| **Blast radius** | Scope of impact when a component fails — design to contain it |
| **Graceful degradation** | System keeps partially working when a dependency fails, rather than total outage |
| **Thundering herd** | Mass simultaneous retries/requests after recovery, overwhelming the system |
| **Connection timeout vs read timeout** | Time to establish connection vs. time to receive data once connected |
| **Idempotency key** | Ensures retried requests don't duplicate side effects |
| **Noisy neighbor** | One tenant/client's load degrading others' performance in a shared system |

## 9. Async / Job Patterns

| Term | Meaning |
|---|---|
| **Job queue (SQS/Kafka)** | Decouple request submission from processing for long-running tasks |
| **Polling vs Webhook** | Client checks job status repeatedly vs. server pushes completion event |
| **Pub-sub fan-out** | One event triggers notification to many subscribers without direct loops |

