# ruvector-router-core

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-router-core**

## Module Description

Core vector database and neural routing inference engine. This module provides the foundation for intelligent request routing using semantic similarity and neural network models, enabling dynamic load balancing and optimal resource allocation.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and router orchestration
- **Vector Database**: `src/vector_db.rs` - Vector storage and retrieval
- **Distance Metrics**: `src/distance.rs` - Similarity calculations
- **Storage**: `src/storage.rs` - Persistent storage layer
- **Index**: `src/index.rs` - HNSW indexing
- **Quantization**: `src/quantization.rs` - Vector compression

## Architecture

### Core Components

1. **Vector Database** (`vector_db.rs`)
   - Vector collection management
   - CRUD operations for vectors
   - Metadata filtering
   - Batch operations

2. **Distance Metrics** (`distance.rs`)
   - Euclidean distance
   - Cosine similarity
   - Dot product
   - Inner product
   - SIMDD-accelerated implementations

3. **Storage Layer** (`storage.rs`)
   - Redb-based persistent storage
   - Memory-mapped files
   - Write-ahead logging
   - Transaction support

4. **HNSW Index** (`index.rs`)
   - Hierarchical navigable small world
   - Approximate nearest neighbor search
   - Dynamic graph construction
   - EfConstruction/efSearch tuning

5. **Quantization** (`quantization.rs`)
   - Product quantization
   - Scalar quantization
   - Binary quantization
   - Compression/decompression

6. **Type System** (`types.rs`)
   - Vector types
   - Metadata types
   - Query types
   - Result types

7. **Error Handling** (`error.rs`)
   - Error types
   - Error conversion
   - Diagnostics

## Key Dependencies

- `redb` - Embedded key-value storage
- `memmap2` - Memory-mapped files
- `rayon` - Parallel processing
- `crossbeam` - Concurrent data structures
- `rkyv` - Zero-copy serialization
- `bincode` - Binary serialization
- `serde` - JSON serialization
- `simsimd` - SIMD distance calculations
- `ndarray` - N-dimensional arrays
- `uuid` - Unique identifiers
- `chrono` - Time handling

## Features

This module is feature-flagged via dependencies and compilation options:
- **SIMD**: Enabled via `simsimd` dependency
- **Parallel**: Enabled via `rayon` and `crossbeam` dependencies
- **Persistence**: Enabled via `redb` and `memmap2` dependencies

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Benchmarks**:
  - `benches/vector_search.rs` - Vector search performance benchmarks

## Platform Bindings

This is a core library that is used by platform-specific bindings:
- **Node.js**: `ruvector-router-ffi` - FFI bindings for Node.js
- **WASM**: `ruvector-router-wasm` - WebAssembly bindings
- **CLI**: `ruvector-router-cli` - Command-line interface

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Graph neural networks
- [ruvector-attention](../ruvector-attention/CLAUDE.md) - Attention mechanisms

## Usage Examples

```rust
use ruvector_router_core::{VectorDB, DistanceMetric};

// Create a new vector database
let db = VectorDB::new_memory()?;

// Insert vectors
db.insert("route-1", &[0.1, 0.2, 0.3], &metadata)?;

// Search for similar vectors
let results = db.search(
    &[0.15, 0.25, 0.35],
    DistanceMetric::Cosine,
    10  // top-k
)?;

// Use for routing decisions
let best_route = results.first()?;
```

## Performance Characteristics

- **Index Build**: O(n log n) for HNSW construction
- **Search**: O(log n) approximate nearest neighbor
- **Insert**: O(log n) amortized
- **Distance**: SIMD-accelerated (10-100x faster than scalar)
- **Storage**: Memory-mapped for zero-copy access

## Routing Applications

This router core is designed for:
1. **Semantic Routing** - Route requests based on semantic similarity
2. **Neural Inference** - Use neural models for routing decisions
3. **Load Balancing** - Distribute requests based on vector embeddings
4. **A/B Testing** - Route traffic for experiments
5. **Feature Flagging** - Route based on feature vectors

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 8 core source files
- Identified key dependencies and features
- Catalogued vector search benchmarks
