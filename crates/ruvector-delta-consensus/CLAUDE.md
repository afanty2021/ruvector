# ruvector-delta-consensus

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-delta-consensus**

## Module Description

Distributed delta consensus using CRDTs (Conflict-free Replicated Data Types) and causal ordering, enabling consistent state synchronization across distributed nodes without coordination for common operations.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and delta consensus
- **Delta Integration**: Uses `ruvector-delta-core` for delta representation

## Architecture

### Core Components

1. **CRDT Operations**
   - Delta-CRDT (δ-CRDT) implementation
   - Causal ordering
   - Conflict resolution
   - State convergence

2. **Consensus Protocols**
   - Causal consistency
   - Eventual consistency
   - Conflict-free merges
   - Vector clocks

3. **Delta Propagation**
   - Efficient delta dissemination
   - Anti-entropy
   - State synchronization
   - Bulk transfer optimization

## Key Dependencies

- `ruvector-delta-core` - Core delta types and traits
- `thiserror` - Error handling
- `parking_lot` - Fast mutexes
- `dashmap` - Concurrent hashmap
- `smallvec` - Small vector optimization
- `serde` - JSON serialization
- `bincode` - Binary serialization
- `uuid` - Unique identifiers
- `chrono` - Time handling
- `tokio` - Async runtime (optional)

## Features

- **default** - Basic functionality
- **async** - Enable async runtime support with Tokio

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Property Tests**: Use `proptest` for property-based testing
- **Benchmarks**: Planned but not yet implemented

## Usage Examples

```rust
use ruvector_delta_consensus::{DeltaCRDT, ReplicaId};

// Create replicas
let replica_a = DeltaCRDT::new(ReplicaId::from("a"));
let replica_b = DeltaCRDT::new(ReplicaId::from("b"));

// Apply local update to replica A
replica_a.apply_local_update(delta)?;

// Generate delta for replication
let delta = replica_a.generate_delta()?;

// Apply delta to replica B
replica_b.apply_remote_delta(delta)?;

// State converges without coordination
assert_eq!(replica_a.state(), replica_b.state());
```

## CRDT Types

1. **Delta-CRDT (δ-CRDT)**
   - Efficient state-based replication
   - Compact delta representation
   - Optimal dissemination

2. **Causal CRDT**
   - Causal ordering
   - Vector clock tracking
   - Happens-before relations

3. **State-based CRDT**
   - Full state exchange
   - Merge operation
   - Idempotent updates

## Consistency Models

1. **Causal Consistency**
   - Preserve causal relationships
   - No concurrent updates lost

2. **Eventual Consistency**
   - Converge to same state
   - No coordination required

3. **Strong Eventual Consistency**
   - Converge to same state
   - Conflict-free resolution

## Performance Characteristics

- **Local Update**: O(1) for single operation
- **Delta Generation**: O(k) where k = operations count
- **Merge**: O(n) where n = state size
- **Conflict Resolution**: O(k) where k = conflicts

## Use Cases

1. **Distributed Databases** - Multi-master replication
2. **Collaborative Editing** - Real-time collaboration
3. **Edge Computing** - Offline-first applications
4. **IoT Networks** - Distributed sensor data
5. **Social Networks** - Timeline synchronization

## Related Modules

- [ruvector-delta-core](../ruvector-delta-core/CLAUDE.md) - Core delta types
- [ruvector-delta-index](../ruvector-delta-index/CLAUDE.md) - Delta-aware HNSW
- [ruvector-delta-graph](../ruvector-delta-graph/CLAUDE.md) - Graph deltas
- [ruvector-raft](../ruvector-raft/CLAUDE.md) - Strong consensus

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented CRDT operations and consensus protocols
- Identified consistency models and features
- Catalogued use cases and performance
