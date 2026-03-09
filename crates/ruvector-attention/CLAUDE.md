[根目录](../../CLAUDE.md) > **ruvector-attention**

# ruvector-attention - Module Documentation

## Module Responsibilities

46 attention mechanisms grounded in 7 mathematical theories. From Flash Attention to optimal transport, mincut-gated transformers to hyperbolic attention.

## Entry Points

- **`src/lib.rs`** - Main library entry point
- **`src/sdk/`** - High-level SDK with builder API
- **`src/attention/`** - Standard attention (scaled dot-product, multi-head)
- **`src/sparse/`** - Sparse attention (Flash, linear, local-global)
- **`src/hyperbolic/`** - Hyperbolic geometry attention
- **`src/transport/`** - Optimal transport attention
- **`src/topology/`** - Topology-gated attention

## Key Interfaces

### SDK Builder API
```rust
// Simple multi-head attention
let attention = multi_head(768, 12)
    .dropout(0.1)
    .causal(true)
    .build()?;

// Flash attention for long sequences
let flash = flash(1024, 128)
    .causal(true)
    .build()?;

// Hyperbolic attention
let hyperbolic = hyperbolic(512, -1.0)
    .build()?;
```

### Core Trait
```rust
pub trait Attention {
    fn compute(&self, query: &[f32], keys: &[&[f32]], values: &[&[f32]]) -> Result<Vec<f32>>;
    fn forward(&self, x: &Array2<f32>) -> Result<Array2<f32>>;
}
```

## 7 Mathematical Theories

1. **Optimal Transport** (`src/transport/`) - Sliced Wasserstein, Centroid OT
2. **Mixed Curvature** (`src/curvature/`) - E+H+S product manifolds
3. **Topology Gating** (`src/topology/`) - Coherence-based switching
4. **Information Geometry** (`src/info_geometry/`) - Fisher metric, natural gradient
5. **Information Bottleneck** (`src/info_bottleneck/`) - KL compression
6. **PDE/Diffusion** (`src/pde_attention/`) - Heat equation on graph
7. **Unified Diagnostics** (`src/unified_report/`) - Health monitoring

## Dependencies

### Required
- `rayon` - Data parallelism
- `thiserror` - Error handling
- `serde` - Serialization

### Optional
- `ruvector-math` - Advanced math primitives
- `napi` - Node.js bindings

## Testing

### Test Status
- **142 passing tests**

### Running Tests
```bash
# All tests
cargo test -p ruvector-attention

# Benchmarks
cargo bench -p ruvector-attention

# Specific attention type
cargo test -p ruvector-attention test_flash_attention
```

## Key Files

| Directory | Description |
|-----------|-------------|
| `src/sdk/` | High-level builder API |
| `src/attention/` | Standard attention |
| `src/sparse/` | Flash, linear, local-global |
| `src/hyperbolic/` | Hyperbolic, mixed curvature |
| `src/graph/` | GAT, RoPE |
| `src/moe/` | Mixture-of-Experts |
| `src/transport/` | Optimal transport |
| `src/topology/` | Topology-gated |
| `src/info_geometry/` | Fisher, natural gradient |
| `src/info_bottleneck/` | KL, VIB |
| `src/pde_attention/` | Diffusion |
| `src/unified_report/` | Diagnostics |

## Supported Attention Mechanisms

### Standard
- Scaled Dot-Product
- Multi-Head

### Sparse (O(n) memory)
- Flash Attention
- Linear Attention
- Local-Global

### Geometric
- Hyperbolic Attention
- Mixed Curvature

### Graph
- Edge-Featured GAT
- RoPE for Graphs

### MoE
- Mixture-of-Experts
- Top-k Routing

## Performance

### Complexity Comparison

| Mechanism | Time | Memory | Use Case |
|-----------|------|--------|----------|
| Scaled Dot-Product | O(n²) | O(n²) | Short sequences |
| Flash Attention | O(n²) | O(n) | Long sequences |
| Linear Attention | O(n) | O(n) | Very long sequences |
| Local-Global | O(n·w) | O(n·w) | Documents |
| Hyperbolic | O(n²) | O(n²) | Hierarchies |
| MoE | O(n²/E) | O(n²) | Specialized tasks |

### Benchmarks
(batch_size=32, seq_len=512, dim=768)
- Flash: 2.3x faster, 5x less memory
- Linear: O(n) scaling >4096
- Local-Global: 60% of standard (w=256)
- Sliced Wasserstein: 1.8x slower, better matching

## Feature Flags

- `simd` (default) - SIMD acceleration
- `wasm` - WebAssembly support
- `napi` - Node.js bindings
- `math` - Advanced math (OT, mixed curvature)
- `sheaf` - Sheaf attention (ADR-015)

## Common Patterns

### Transformer Block
```rust
use ruvector_attention::sdk::*;

let attention = multi_head(dim, 12)
    .dropout(0.1)
    .build()?;

let pipeline = AttentionPipeline::new()
    .add_norm(NormType::LayerNorm)
    .add_attention(attention)
    .add_dropout(0.1)
    .add_residual();
```

### Adaptive Attention
```rust
let report = ReportBuilder::new(ReportConfig::default())
    .analyze_keys(&keys)
    .build();

match report.recommendation {
    AttentionRecommendation::Standard => { /* ... */ }
    AttentionRecommendation::Sparse => { /* ... */ }
    AttentionRecommendation::Geometric => { /* ... */ }
    AttentionRecommendation::Diffusion => { /* ... */ }
}
```

### Presets
```rust
let bert = AttentionPreset::Bert.builder(768).build()?;
let gpt = AttentionPreset::Gpt.builder(768).build()?;
let longformer = AttentionPreset::Longformer.builder(512).build()?;
```

## Related Modules

- **[ruvector-core](../ruvector-core/)** - Vector database
- **[ruvector-gnn](../ruvector-gnn/)** - Graph neural networks
- **[ruvector-graph](../ruvector-graph/)** - Graph database
- **[ruvector-math](../ruvector-math/)** - Math primitives

## Changelog

### 2026-03-09
- Initial module documentation generated
- Documented 46 attention mechanisms
- Identified 7 mathematical theories
- Catalogued SDK patterns
