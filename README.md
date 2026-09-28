# log-compaction-optimiziation-lsm
Optimizing Log Compaction with Log-Structured Merge-tree


## System Architecture Blueprint


To prove deep systems capability, the project shouldn't just wrap RocksDB; it should implement a custom, decoupled, pluggable scheduler via its worker thread pool, or serve as a standalone minimal [LSM-Engine](https://github.com/Logic-HQ/log-compaction-optimiziation-lsm/wiki/LSM%E2%80%90Tree-Engine) written in Rust or C++.

```
       [ Client Write Path ]              [ Active I/O Metrics Telemetry ]
                │                                       │
                ▼                                       ▼
     ┌────────────────────┐                   ┌───────────────────┐
     │  MemTable / WAL    │                   │   io_uring / eBPF │
     └──────────┬─────────┘                   └─────────┬─────────┘
                │ (Flush)                               │
                ▼                                       ▼
     ┌────────────────────┐                   ┌───────────────────┐
     │   Level 0 SSTs     │                   │  Feedback Loop    │
     └────────────────────┘                   └─────────┬─────────┘
                │                                       │ (Throttling / Budget)
                ▼                                       ▼
     ┌────────────────────────────────────────────────────────────┐
     │                Dynamic Compaction Scheduler                │
     │  - Size-Tiered vs Leveled Evaluator                        │
     │  - Thread Pool Control & IOPS Token Bucket                 │
     │  - Overlap Matrix / Priority Queue                         │
     └──────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
                   ┌────────────────────────┐
                   │  Parallel Compaction   │
                   │  - Chunked Iterators   │
                   └────────────────────────┘
```

------------------------------

## [Core Engineering Challenges & Points](https://github.com/Logic-HQ/log-compaction-optimiziation-lsm/wiki)


1. Telemetry Capture: Dynamic I/O & Latency Feedback Loop
2. The Pluggable Scheduler: Hybrid Size-Tiered vs. Leveled
3. Advanced Priority Queuing (The "What to Merge" Problem)
4. Execution Layer: Memory-Efficient Zero-Copy Merging
5. Interacting with an existing database engine like [RocksDB](https://github.com/Logic-HQ/log-compaction-optimiziation-lsm/wiki/Engineers-choose-Google's-Level%E2%80%90DB).
