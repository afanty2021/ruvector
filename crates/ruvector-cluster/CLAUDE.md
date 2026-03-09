# ruvector-cluster

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-cluster**

## Module Description

Distributed clustering and sharding for ruvector, providing consistent hashing, cluster membership, and data distribution primitives for building scalable, fault-tolerant vector and graph databases.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and cluster management
- **Sharding**: `src/shard.rs` - Consistent hashing and data partitioning
- **Discovery**: `src/discovery.rs` - Node discovery and membership
- **Consensus**: `src/consensus.rs` - Cluster consensus mechanisms

## Architecture

### Core Components

1. **Sharding** (`shard.rs`)
   - Consistent hashing (ring-based)
   - Virtual nodes for even distribution
   - Shard allocation and rebalancing
   - Data migration support
   - Fault tolerance and replication

2. **Discovery** (`discovery.rs`)
   - Node discovery protocols
   - Membership management
   - Health checking
   - Failure detection
   - Gossip dissemination

3. **Consensus** (`consensus.rs`)
   - Cluster-wide decision making
   - Configuration agreement
   - Leader election (optional)
   - Quorum management

## Key Dependencies

- `ruvector-core` - Core vector operations
- `tokio` - Async runtime
- `serde` - Serialization
- `serde_json` - JSON serialization
- `bincode` - Binary serialization
- `dashmap` - Concurrent hashmap
- `parking_lot` - Fast mutexes
- `uuid` - Unique identifiers
- `chrono` - Time handling
- `futures` - Async utilities
- `rand` - Random number generation
- `async-trait` - Async trait support

## Features

This module provides core clustering functionality without extensive feature flags. All features are enabled by default for maximum functionality.

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Integration Tests**: Test cluster behavior with multiple nodes

## Usage Examples

```rust
use ruvector_cluster::{Cluster, Shard};

// Create a new cluster
let cluster = Cluster::new("my-cluster")?;

// Add nodes to the cluster
cluster.add_node("node-1", "127.0.0.1:8080").await?;
cluster.add_node("node-2", "127.0.0.1:8081").await?;

// Determine which shard owns a key
let shard = cluster.locate_shard(b"my-vector-id")?;

// Get nodes responsible for a shard
let nodes = cluster.get_shard_nodes(&shard)?;

// Handle node failure
cluster.mark_node_down("node-1").await?;
```

## Cluster Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Cluster Manager                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │   Node   │  │   Node   │  │   Node   │  │   Node   │ │
│  │     1    │  │     2    │  │     3    │  │     4    │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│       │             │             │             │        │
│       └─────────────┴─────────────┴─────────────┘        │
│                     Consistent Hash Ring                  │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐      │
│  │ V1  │ │ V2  │ │ V3  │ │ V4  │ │ V5  │ │ V6  │ ...  │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘      │
└─────────────────────────────────────────────────────────┘
```

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-raft](../ruvector-raft/CLAUDE.md) - Raft consensus
- [ruvector-replication](../ruvector-replication/CLAUDE.md) - Multi-master replication
- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Distributed graph database

## Distributed Features

This module enables:
1. **Horizontal Scaling** - Add nodes to increase capacity
2. **Fault Tolerance** - Continue operating despite node failures
3. **Data Distribution** - Even distribution across nodes
4. **Rebalancing** - Automatic data migration when nodes join/leave
5. **Discovery** - Automatic node discovery and membership

## Performance Characteristics

- **Shard Lookup**: O(log n) for consistent hashing
- **Node Discovery**: O(n) for gossip, O(log n) for structured
- **Rebalancing**: O(k) where k = average items per shard
- **Fault Detection**: O(1) per failure detector

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 4 core source files
- Identified cluster architecture and features
- Catalogued dependencies and use cases
