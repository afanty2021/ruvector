# ruvector-mincut

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-mincut**

## Module Description

World's first subpolynomial dynamic min-cut algorithm providing self-healing networks, AI optimization, and real-time graph analysis. This module implements advanced graph partitioning algorithms with subpolynomial time complexity (n^o(1)), enabling unprecedented scalability for network analysis and optimization.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and algorithm orchestration
- **Core Algorithms**: `src/algorithm/mod.rs` - Approximate and exact min-cut algorithms
- **SNN Engine**: `src/snn/mod.rs` - Spiking Neural Network integration
- **J-Tree**: `src/jtree/mod.rs` - Hierarchical decomposition (ADR-002)

## Architecture

### Core Algorithms (`src/algorithm/`)

1. **Approximate Min-Cut** (`approximate.rs`)
   - (1+ε)-approximation algorithm
   - Subpolynomial time complexity
   - Suitable for large-scale graphs

2. **Replacement Algorithm** (`replacement.rs`)
   - Dynamic edge replacement
   - Self-healing network support

### Subpolynomial Algorithms (`src/subpolynomial/`)

- Breakthrough n^o(1) time complexity
- Applies to sparse and dense graphs
- Theoretical foundations in research papers

### J-Tree Hierarchy (`src/jtree/`)

1. **Coordinator** (`coordinator.rs`)
   - Two-tier coordination
   - Load balancing

2. **Hierarchy** (`hierarchy.rs`)
   - Multi-level tree structure
   - Hierarchical decomposition

3. **Level** (`level.rs`)
   - Per-level operations
   - Inter-level communication

4. **Sparsifier** (`sparsifier.rs`)
   - Graph sparsification
   - Cut approximation

### Spiking Neural Networks (`src/snn/`)

1. **Cognitive Engine** (`cognitive_engine.rs`)
   - Neural optimization
   - Adaptive learning

2. **Attractor Dynamics** (`attractor.rs`)
   - Temporal attractors
   - Convergence analysis

3. **Strange Loop** (`strange_loop.rs`)
   - Self-referential optimization
   - Meta-learning

4. **Time Crystal** (`time_crystal.rs`)
   - Periodic structure discovery
   - Temporal patterns

5. **Morphogenetic** (`morphogenetic.rs`)
   - Growth patterns
   - Self-organization

6. **Causal Discovery** (`causal.rs`)
   - Causal inference
   - Network structure learning

### Connectivity (`src/connectivity/`)

1. **Polylogarithmic** (`polylog.rs`)
   - O(log^k n) connectivity testing
   - Efficient sparse graph operations

2. **Cache Optimization** (`cache_opt.rs`)
   - Memory-efficient algorithms
   - Cache-aware data structures

### Witness System (`src/witness/`, `src/instance/`)

1. **Witness** (`witness.rs`)
   - Proof generation
   - Certificate verification

2. **Instance Management** (`instance/mod.rs`)
   - Graph instances
   - Bounded instances
   - Stub instances

### Optimization (`src/optimization/`)

1. **SIMD Distance** (`simd_distance.rs`)
   - SIMD-accelerated distance calculations
   - Batch processing

2. **Parallel** (`parallel.rs`)
   - Multi-threaded execution
   - Work stealing

3. **DSPAR** (`dspar.rs`)
   - Dynamic sparse approximate routing
   - Network optimization

4. **WASM Batch** (`wasm_batch.rs`)
   - WASM-optimized batch operations
   - Browser execution

### WASM Support (`src/wasm/`)

1. **Agentic** (`agentic.rs`)
   - 256-core parallel processing
   - Agent-based computation

2. **SIMD** (`simd.rs`)
   - SIMD operations for WASM

### Canonical Min-Cut (`src/canonical/`)

- Pseudo-deterministic canonical min-cut
- Cactus representation
- Unique cut identification

## Key Dependencies

- `ruvector-core` - Core vector operations (optional, for integration)
- `ruvector-graph` - Graph database integration (optional)
- `petgraph` - Graph data structures
- `rayon` - Parallel processing
- `roaring` - Compressed bitmaps

## Features

- **exact** - Exact minimum cut algorithm
- **approximate** - (1+ε)-approximate algorithm
- **integration** - GraphDB integration
- **monitoring** - Real-time monitoring with callbacks
- **simd** - SIMD optimizations
- **wasm** - WASM compatibility
- **agentic** - 256-core parallel agentic backend
- **jtree** - J-Tree hierarchical decomposition (ADR-002)
- **tiered** - Two-tier coordinator (j-tree + exact)
- **canonical** - Pseudo-deterministic canonical min-cut
- **all-cut-queries** - Sparsest cut, multiway, multicut queries

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Benchmarks**:
  - `benches/mincut_bench.rs` - Core algorithm benchmarks
  - `benches/bounded_bench.rs` - Bounded instance benchmarks
  - `benches/sota_bench.rs` - State-of-the-art comparisons
  - `benches/paper_algorithms_bench.rs` - Research paper reproductions
  - `benches/snn_bench.rs` - Spiking neural network benchmarks
  - `benches/jtree_bench.rs` - J-Tree hierarchy benchmarks
  - `benches/optimization_bench.rs` - Optimization benchmarks

## Examples

- **Temporal Attractors**: `examples/mincut/temporal_attractors/` - Temporal pattern discovery
- **Strange Loop**: `examples/mincut/strange_loop/` - Self-referential optimization
- **Causal Discovery**: `examples/mincut/causal_discovery/` - Causal inference
- **Time Crystal**: `examples/mincut/time_crystal/` - Periodic structure analysis
- **Morphogenetic**: `examples/mincut/morphogenetic/` - Growth pattern simulation
- **Neural Optimizer**: `examples/mincut/neural_optimizer/` - Neural network optimization
- **Benchmarks**: `examples/mincut/benchmarks/` - Performance benchmarks
- **Subpoly Bench**: `examples/mincut/subpoly_bench/` - Subpolynomial algorithm testing

## Platform Bindings

- **Node.js**: `ruvector-mincut-node` - NAPI-RS bindings
- **WASM**: `ruvector-mincut-wasm` - WebAssembly bindings
- **Gated Transformer**: `ruvector-mincut-gated-transformer` - Gated transformer integration
- **Attention**: `ruvector-attn-mincut` - Attention mechanism integration

## Related Modules

- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database engine
- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Graph neural networks
- [ruvector-cluster](../ruvector-cluster/CLAUDE.md) - Distributed clustering

## Performance Characteristics

- **Time Complexity**: Subpolynomial n^o(1) for key algorithms
- **Space Complexity**: Linear O(n + m) for sparse graphs
- **Parallelism**: Multi-core scaling with Rayon
- **WASM**: Optimized for browser execution with agentic backend

## Research Foundation

This module implements cutting-edge algorithms from academic research:
- Subpolynomial min-cut algorithms
- Spiking neural network optimization
- Hierarchical graph decomposition (J-Tree)
- Causal discovery in networks
- Temporal attractor dynamics

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 60+ source files across 10 major components
- Identified key algorithms and features
- Catalogued 8 examples and 7 benchmark suites
