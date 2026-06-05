# Ideas & Concepts
A collection of low-level systems design ideas compilers, databases, and serialization.

### [Modern Compiler](ideas/modernCompiler/README.MD)

An extensible compiler architecture built around three phases: parsing, enrichment, lowering.
Key idea: extensible parser and embedded resolution via dependency graphs for enrichments.

### [RPC](ideas/rpc/README.MD)

A lightweight remote procedure call system where local functions are transparently serialized into binary frames and dispatched over a channel.

### [Serialization](ideas/serialization/README.MD)

A compact binary format that serializes existing structs with refs directly, no wrapper classes needed.

### [Universal DB](ideas/universalDB/README.MD)

A high-performance, embeddable, adaptive NoSQL database with a Linux-kernel-style modular architecture.
Design goal: selective compilation for minimal base builds, runtime extension for everything else.

