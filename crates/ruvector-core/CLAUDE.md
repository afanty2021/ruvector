[根目录](../../CLAUDE.md) > **ruvector-core**

# ruvector-core - Module Documentation

## Module Responsibilities

Foundation vector database engine providing high-performance vector storage, HNSW indexing, quantization, and SIMD acceleration. This is the core library that powers the entire RuVector ecosystem.

## Entry Points

- **`src/lib.rs`** - Main library entry point
- **`src/vector_db.rs`** - `VectorDB` struct, primary API
- **`src/index/hnsw.rs`** - HNSW index implementation
- **`src/quantization.rs`** - Scalar, Product, and Binary quantization

## Key Interfaces

### Core Types
```rust
pub struct VectorDB { /* ... */ }
pub struct VectorEntry { /* ... */ }
pub struct SearchQuery { /* ... */ }
pub struct SearchResult { /* ... */ }
pub enum DistanceMetric { Euclidean, Cosine, DotProduct, Manhattan }
```

### Main Operations
```rust
impl VectorDB {
    pub fn new(options: DbOptions) -> Result<Self>;
    pub fn insert(&self, entry: VectorEntry) -> Result<VectorId>;
    pub fn insert_batch(&self, entries: Vec<VectorEntry>) -> Result<Vec<VectorId>>;
    pub fn search(&self, query: SearchQuery) -> Result<Vec<SearchResult>>;
    pub fn delete(&self, id: &str) -> Result<bool>;
    pub fn get(&self, id: &str) -> Result<Option<VectorEntry>>;
}
```

## Dependencies

### Core Dependencies
- `redb` 2.1 - Embedded database storage
- `hnsw_rs` 0.3 - HNSW indexing (patched for WASM)
- `simsimd` 5.9 - SIMD distance calculations
- `rayon` 1.10 - Parallel processing
- `ndarray` 0.16 - N-dimensional arrays
- `nalgebra` 0.33 - Linear algebra

### Internal Dependencies
- `ruvector-math` - Advanced math operations
- `ruvector-attention` - Attention mechanisms

## Data Models

### VectorEntry
```rust
pub struct VectorEntry {
    pub id: Option<VectorId>,
    pub vector: Vec<f32>,
    pub metadata: Option<HashMap<String, serde_json::Value>>,
}
```

### SearchQuery
```rust
pub struct SearchQuery {
    pub vector: Vec<f32>,
    pub k: usize,
    pub filter: Option<HashMap<String, serde_json::Value>>,
    pub ef_search: Option<usize>,
}
```

### QuantizationConfig
```rust
pub enum QuantizationConfig {
    Scalar,           // 4x compression
    Product {         // 8-32x compression
        subspaces: usize,
        k: usize,
    },
    Binary,           // 32x compression
}
```

## Testing

### Test Structure
- Unit tests in `src/` alongside implementation
- Integration tests in `tests/` directory
- Benchmarks in `benches/` directory

### Running Tests
```bash
# All tests
cargo test -p ruvector-core

# With features
cargo test -p ruvector-core --features simd

# Specific test
cargo test -p ruvector-core test_insert

# Benchmarks
cargo bench -p ruvector-core
```

### Available Benchmarks
- `distance_metrics` - SIMD-optimized distance calculations
- `hnsw_search` - HNSW index search performance
- `quantization_bench` - Quantization techniques
- `batch_operations` - Batch insert/search operations
- `comprehensive_bench` - Full system benchmarks

## Key Files

| File | Description |
|------|-------------|
| `src/lib.rs` | Main library exports |
| `src/vector_db.rs` | VectorDB implementation |
| `src/index/hnsw.rs` | HNSW index wrapper |
| `src/quantization.rs` | Quantization methods |
| `src/distance.rs` | Distance metrics |
| `src/storage.rs` - `src/storage_memory.rs` | Storage backends |
| `src/advanced_features/` | Advanced search features |
| `src/advanced/` | Hypergraph, learned index, TDA |

## Performance Characteristics

### Latency
- Search (1K vectors): ~0.1ms (flat), ~0.2ms (HNSW)
- Search (100K vectors): ~10ms (flat), ~0.5ms (HNSW)
- Search (1M vectors): ~100ms (flat), <1ms (HNSW)
- Insert: ~0.1ms (flat), ~1ms (HNSW)

### Memory (1M vectors, 384 dimensions)
- Full Precision: ~1.5GB
- Scalar Quantization: ~400MB (98% recall)
- Product Quantization: ~200MB (95% recall)
- Binary Quantization: ~50MB (85% recall)

### Throughput
- Single Thread: ~2,000 QPS
- Multi-Thread (8 cores): ~50,000 QPS
- With SIMD: ~80,000 QPS

## Feature Flags

- `simd` (default) - SIMD acceleration via SimSIMD
- `parallel` (default) - Parallel processing via Rayon
- `storage` (default) - File-based storage via redb
- `hnsw` (default) - HNSW indexing
- `memory-only` - Pure in-memory storage for WASM
- `api-embeddings` - API-based embeddings

## Common Patterns

### Creating a Database
```rust
let db = VectorDB::new(DbOptions {
    dimensions: 384,
    distance_metric: DistanceMetric::Cosine,
    storage_path: "./vectors.db".to_string(),
    ..Default::default()
})?;
```

### Inserting Vectors
```rust
db.insert(VectorEntry {
    id: Some("doc1".to_string()),
    vector: vec![0.1, 0.2, 0.3, /* ... */],
    metadata: Some(HashMap::from([
        ("title".to_string(), json!("Document 1"))
    ])),
})?;
```

### Searching
```rust
let results = db.search(SearchQuery {
    vector: query_vector,
    k: 10,
    filter: Some(HashMap::from([
        ("category".to_string(), json!("tech"))
    ])),
    ef_search: Some(100),
})?;
```

## Related Modules

- **[ruvector-gnn](../ruvector-gnn/)** - GNN layer for learning
- **[ruvector-attention](../ruvector-attention/)** - Attention mechanisms
- **[ruvector-node](../ruvector-node/)** - Node.js bindings
- **[ruvector-wasm](../ruvector-wasm/)** - WASM bindings

## Changelog

### 2026-03-09
- Initial module documentation generated
- Identified key interfaces and dependencies
- Documented performance characteristics
