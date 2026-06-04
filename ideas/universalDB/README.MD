# Universal DB

A high-performance, embeddable, adaptive NoSQL database with a modular architecture inspired by the Linux kernel — some modules are compiled into the core, others loaded at runtime.

## Key Requirements

- Maximum performance
- NoSQL
- Adaptive
- Embeddable

## Architecture

Modules are split into **core** (always present) and **dynamic** (loaded at runtime). Core modules are mandatory for basic operation; dynamic modules extend functionality without bloating the base build.

## Modules

### Memory
Defines how data is stored and retrieved. Supports multiple storage backends (RAM, Disk, Flash, etc.).

**Interface:** `read`, `write`

### Transactions *(optional)*
Runs transactions with rollback support. Every mutable operation must have a corresponding revert function (e.g. `deleteUser` → moves record to temp storage; on failure, `revertDeleteUser` restores it).

### Modification *(core only)*
Handles structural changes to data — enables dynamic schema modifications and migrations at runtime.

### Scripting *(optional)*
Executes query scripts (e.g. SQL-like syntax) supporting `add`, `update`, `find`, and similar operations.

### Distributed *(optional)*
Enables distributed queries. Can sync with a master node or partition data across nodes.

### RPC *(optional)*
Defines how data is transferred between client and server. Can be embedded in core or loaded dynamically.

### Cache *(optional)*
Caches results of read-only operations (e.g. `getUserById`). Can be core or dynamic.

### Security *(optional)*
Enforces access control, authentication, and ACL checks on database operations.

### Migrations *(optional)*
Manages versioned schema changes, coordinating with the Modification module.

### Replication *(optional)*
Replicates data across nodes for redundancy and availability.
