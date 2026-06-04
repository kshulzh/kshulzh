# kshulzh — Ideas & Concepts

A collection of low-level systems design ideas spanning compilers, databases and serialization.

---

## Projects

### [Modern Compiler](ideas/modernCompiler/README.MD)

An extensible compiler architecture built around three incremental phases:

1. **Parsing** — source text → unenriched IR via composable, overridable parsers. Grammar extensions (e.g. new keywords) are added without modifying the core.
2. **Enrichment** — lazy symbol resolution using a *problem graph*: each unresolved reference is a `ProblemNode` that resolves itself on demand, naturally handling dependency chains.
3. **Lowering** — visitor walk over the enriched IR emitting target output (e.g. bytecode).

Key idea: incremental compilation and pluggable grammar make the compiler cheap to extend without forking.

---

### [RPC](ideas/rpc/README.MD)

A lightweight remote procedure call system where local functions are transparently serialized into binary frames and dispatched over a channel.

```
struct rpc  { int functionId; args args; }
struct args { int size; stream *data; }
```

A router maps function IDs to handlers. Callbacks are implemented as **reverse RPC frames** sent back to the caller over the same channel, enabling bidirectional streaming without extra infrastructure.

---

### [Serialization](ideas/serialization/README.MD)

A compact binary format that serializes existing structs directly — no wrapper classes needed.

- **Flat structs** — fields written sequentially; variable-length fields prefixed with their size.
- **Graph/cyclic structures** — references stored as byte-array offsets instead of inline data, handling cycles naturally.
- **Inheritance** — class IDs distinguish subtypes at deserialization time.
- **Meta language** — concise type notation: `i4` (int), `f4` (float), `P<T>` (pointer), `a<T>` (array).

---

### [Universal DB](ideas/universalDB/README.MD)

A high-performance, embeddable, adaptive NoSQL database with a Linux-kernel-style modular architecture.

**Core modules** (always compiled in): memory backends (RAM/Disk/Flash), dynamic schema modification.

**Dynamic modules** (loaded at runtime):

| Module | Purpose |
|---|---|
| Transactions | Rollback via per-operation revert functions |
| Scripting | SQL-like query execution (`add`, `update`, `find`) |
| Distributed | Sharding, master sync, partitioning |
| RPC | Client-server data transfer |
| Cache | Read-only result caching |
| Security | ACL, authentication, access control |
| Migrations | Versioned schema changes |
| Replication | Multi-node data redundancy |

Design goal: selective compilation for minimal base builds, runtime extension for everything else.

---

## Status

All projects are in the **concept / design phase** — documentation and architecture only, no implementation yet.
