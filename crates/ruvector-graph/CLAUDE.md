# ruvector-graph

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-graph**

## Module Description

Distributed Neo4j-compatible hypergraph database with SIMD optimization, Cypher query language support, and hybrid vector-graph capabilities. This module provides a graph database engine that combines traditional graph operations with modern vector similarity search and neural network integration.

## Entry Points

- **Main Library**: `src/lib.rs` - Core graph database API
- **Cypher Parser**: `src/cypher/mod.rs` - Cypher query language parser and executor
- **Graph Operations**: `src/graph.rs` - Core graph data structures
- **Node/Edge**: `src/node.rs`, `src/edge.rs` - Node and edge implementations
- **Hyperedges**: `src/hyperedge.rs` - Hypergraph support (edges connecting multiple nodes)

## Architecture

### Core Components

1. **Graph Storage** (`src/storage.rs`)
   - Redb-based persistent storage
   - Memory-mapped file support
   - Compression options (ZSTD, LZ4)

2. **Cypher Query Engine** (`src/cypher/`)
   - `lexer.rs` - Tokenizes Cypher queries
   - `parser.rs` - Parses Cypher syntax
   - `ast.rs` - Abstract syntax tree
   - `optimizer.rs` - Query optimization
   - `semantic.rs` - Semantic analysis

3. **Query Executor** (`src/executor/`)
   - `operators.rs` - Query execution operators
   - `pipeline.rs` - Query pipeline execution
   - `plan.rs` - Query planning
   - `cache.rs` - Result caching
   - `parallel.rs` - Parallel query execution

4. **Hybrid Features** (`src/hybrid/`)
   - `graph_neural.rs` - GNN integration
   - `semantic_search.rs` - Vector similarity search
   - `rag_integration.rs` - RAG (Retrieval Augmented Generation)
   - `vector_index.rs` - Vector indexing

5. **Distributed Operations** (`src/distributed/`)
   - `shard.rs` - Data sharding
   - `replication.rs` - Multi-master replication
   - `federation.rs` - Cross-cluster federation
   - `gossip.rs` - Gossip protocol
   - `coordinator.rs` - Distributed coordinator

6. **Optimization** (`src/optimization/`)
   - `simd_traversal.rs` - SIMD-accelerated graph traversal
   - `cache_hierarchy.rs` - Multi-level caching
   - `bloom_filter.rs` - Bloom filter indexes
   - `query_jit.rs` - JIT query compilation
   - `adaptive_radix.rs` - Adaptive radix trees

## Key Dependencies

- `ruvector-core` - Vector operations and HNSW indexing
- `ruvector-raft` - Raft consensus (optional, for distributed mode)
- `ruvector-cluster` - Cluster management (optional)
- `ruvector-replication` - Multi-master replication (optional)
- `petgraph` - Graph algorithms
- `redb` - Embedded database
- `simsimd` - SIMD distance calculations
- `rayon` - Parallel processing
- `nom` / `pest` - Parser combinators

## Features

- **simd** - Enable SIMD optimizations for distance calculations
- **storage** - Enable persistent storage backends
- **async-runtime** - Enable async operations with Tokio
- **compression** - Enable data compression (ZSTD, LZ4)
- **distributed** - Enable distributed deployment with RAFT
- **federation** - Enable cross-cluster federation
- **cypher-pest** - Use PEST parser for Cypher
- **cypher-lalrpop** - Use LALRPOP parser for Cypher
- **fulltext** - Enable full-text search
- **geospatial** - Enable geospatial indexing
- **temporal** - Enable temporal graph support

## Testing

- **Unit Tests**: Located in each module's `tests` module
- **Benchmarks**: `benches/new_capabilities_bench.rs`
- **Examples**:
  - `examples/test_cypher_parser.rs` - Cypher parser testing

## Usage Examples

```rust
use ruvector_graph::{GraphStore, QueryExecutor};

// Create a new graph
let graph = GraphStore::new_memory();

// Add nodes and edges
let node1 = graph.add_node("Person", &[("name", "Alice")])?;
let node2 = graph.add_node("Person", &[("name", "Bob")])?;
graph.add_edge(node1, node2, "KNOWS", &[("since", "2020")])?;

// Execute Cypher query
let results = graph.cypher("MATCH (a)-[r:KNOWS]->(b) RETURN a, b")?;
```

## Platform Bindings

- **Node.js**: `ruvector-graph-node` - NAPI-RS bindings
- **WASM**: `ruvector-graph-wasm` - WebAssembly bindings
- **Transformer**: `ruvector-graph-transformer` - Graph transformer operations

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-mincut](../ruvector-mincut/CLAUDE.md) - Graph partitioning algorithms
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Graph neural networks
- [ruvector-raft](../ruvector-raft/CLAUDE.md) - Distributed consensus

## Performance Characteristics

- **Storage**: Memory-mapped files with optional compression
- **Query**: Parallel execution with SIMD optimizations
- **Indexing**: HNSW for vector similarity, custom indexes for properties
- **Distributed**: Sharding with consistent hashing, Raft consensus

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 46 source files across 9 major components
- Identified key features and platform bindings
