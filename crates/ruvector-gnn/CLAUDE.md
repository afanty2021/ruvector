[根目录](../../CLAUDE.md) > **ruvector-gnn**

# ruvector-gnn - Module Documentation

## Module Responsibilities

Graph Neural Network layer that makes HNSW vector search get smarter over time. Operates directly on HNSW graph topology to learn from query patterns and improve search results.

## Entry Points

- **`src/lib.rs`** - Main library entry point
- **`src/layers/`** - GNN layer implementations (GCN, GAT, GraphSAGE)
- **`src/message_passing.rs`** - Message passing interface
- **`src/embedding.rs`** - GNN embedder for HNSW graphs

## Key Interfaces

### Core Types
```rust
pub struct GCNLayer { /* Graph Convolutional Network */ }
pub struct GATLayer { /* Graph Attention Network */ }
pub struct GraphSAGE { /* Inductive learning */ }
pub struct GNNConfig { /* Configuration */ }
```

### Message Passing Trait
```rust
pub trait MessagePassing {
    fn aggregate(&self, features: &Array2<f32>, neighbors: &[Vec<usize>]) -> Array2<f32>;
    fn update(&self, aggregated: &Array2<f32>, self_features: &Array2<f32>) -> Array2<f32>;
    fn forward(&self, features: &Array2<f32>, adjacency: &[Vec<usize>]) -> Result<Array2<f32>>;
}
```

## Dependencies

### Required
- `ruvector-core` 2.0.1 - Vector database engine
- `ndarray` - N-dimensional arrays
- `rayon` - Data parallelism
- `serde` - Serialization

### Optional
- `memmap2` - Memory-mapped weights
- `napi` - Node.js bindings

## Data Models

### GNNConfig
```rust
pub struct GNNConfig {
    pub input_dim: usize,
    pub output_dim: usize,
    pub hidden_dim: usize,
    pub num_heads: usize,     // For GAT
    pub dropout: f32,
    pub activation: Activation,
}
```

### AttentionConfig
```rust
pub struct AttentionConfig {
    pub input_dim: usize,
    pub output_dim: usize,
    pub num_heads: usize,
    pub concat_heads: bool,
    pub dropout: f32,
    pub leaky_relu_slope: f32,
}
```

## Testing

### Running Tests
```bash
# All tests
cargo test -p ruvector-gnn

# With features
cargo test -p ruvector-gnn --features simd,mmap

# Benchmarks
cargo bench -p ruvector-gnn
```

### Test Coverage
- Unit tests for all layer types
- Message passing correctness
- Attention weight computation
- Gradient flow checks

## Key Files

| File | Description |
|------|-------------|
| `src/lib.rs` | Main exports |
| `src/layers/gcn.rs` | GCN layer |
| `src/layers/gat.rs` | GAT layer |
| `src/layers/sage.rs` | GraphSAGE |
| `src/message_passing.rs` | Message passing trait |
| `src/aggregators.rs` - Mean, Max, LSTM |
| `src/embedding.rs` | GNN embedder |

## Performance Characteristics

### Latency (100K nodes, avg degree 16)
- GCN forward (1 layer): ~15ms
- GAT forward (8 heads): ~45ms
- GraphSAGE (2 layers): ~25ms
- Message aggregation: ~5ms

### Memory Usage
- 128 -> 64 (1 layer): ~50MB
- 128 -> 64 (4 layers): ~150MB
- With mmap weights: ~10MB + disk

## Feature Flags

- `simd` (default) - SIMD-optimized operations
- `mmap` (default) - Memory-mapped weight storage
- `wasm` - WebAssembly-compatible build
- `napi` - Node.js bindings
- `cold-tier` - Hyperbatch training for graphs exceeding RAM

## Common Patterns

### Basic GCN Layer
```rust
let config = GNNConfig {
    input_dim: 128,
    output_dim: 64,
    hidden_dim: 128,
    num_heads: 4,
    dropout: 0.1,
    activation: Activation::ReLU,
};

let gcn = GCNLayer::new(config)?;
let output = gcn.forward(&features, &adjacency)?;
```

### Graph Attention
```rust
let config = AttentionConfig {
    input_dim: 128,
    output_dim: 64,
    num_heads: 8,
    concat_heads: true,
    dropout: 0.1,
    leaky_relu_slope: 0.2,
};

let gat = GATLayer::new(config)?;
let (output, attention_weights) = gat.forward_with_attention(&features, &adjacency)?;
```

### Integration with ruvector-core
```rust
use ruvector_core::VectorDB;
use ruvector_gnn::{HNSWMessagePassing, GNNEmbedder};

let db = VectorDB::open("vectors.db")?;
let gnn = GNNEmbedder::new(GNNConfig {
    input_dim: db.dimensions()?,
    output_dim: 64,
    ..Default::default()
})?;

let hnsw_graph = db.get_hnsw_graph()?;
let gnn_embeddings = gnn.encode(&db.get_all_vectors()?, &hnsw_graph)?;
```

## Related Modules

- **[ruvector-core](../ruvector-core/)** - Vector database engine
- **[ruvector-attention](../ruvector-attention/)** - Attention mechanisms
- **[ruvector-graph](../ruvector-graph/)** - Graph database
- **[ruvector-gnn-node](../ruvector-gnn-node/)** - Node.js bindings
- **[ruvector-gnn-wasm](../ruvector-gnn-wasm/)** - WASM bindings

## Changelog

### 2026-03-09
- Initial module documentation generated
- Documented GCN, GAT, GraphSAGE layers
- Identified key patterns and interfaces
