# prime-radiant

[根目录](../../CLAUDE.md) > [crates](../) > **prime-radiant**

## Module Description

Universal coherence engine using sheaf Laplacian mathematics for AI safety, hallucination detection, and structural consistency verification in LLMs and distributed systems. This module implements cutting-edge coherence theory for ensuring consistency and safety in AI systems.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and coherence engine
- **Coherence Computation**: Core sheaf Laplacian operations
- **Safety Verification**: AI safety checks and validation

## Architecture

### Core Concepts

1. **Sheaf Theory**
   - Sheaf Laplacian operators
   - Cohomology computation
   - Consistency verification
   - Restriction maps

2. **Coherence Metrics**
   - Residual computation
   - Energy functionals
   - Consistency scores
   - Coherence gates

3. **AI Safety**
   - Hallucination detection
   - Consistency verification
   - Structural validation
   - Safety guarantees

4. **Integration Points**
   - Cognitive tiles (cognitum-gate-kernel)
   - Sona adaptive thresholds
   - GNN learned restrictions
   - Attention topology gating

## Key Dependencies

### Ecosystem Integration

1. **Cognitive Tiles**
   - `cognitum-gate-kernel` - 256-tile WASM coherence fabric

2. **Adaptive Learning**
   - `ruvector-sona` - Self-optimizing thresholds with EWC++
   - `ruvector-gnn` - Learned restriction maps
   - `ruvector-mincut` - Subpolynomial graph partitioning

3. **Neural Systems**
   - `ruvector-hyperbolic-hnsw` - Hierarchy-aware Poincare energy
   - `ruvector-nervous-system` - Coherence gating and HDC witnesses
   - `ruvector-attention` - Topology-gated attention

4. **Core Infrastructure**
   - `ruvector-core` - Vector storage and HNSW
   - `ruvector-graph` - Graph data structures
   - `ruvector-raft` - Distributed consensus
   - `ruvllm` - LLM serving runtime

### Performance Dependencies

- `ndarray` - N-dimensional arrays
- `nalgebra` - Linear algebra (optional)
- `rayon` - Parallel processing (optional)
- `wide` - SIMD (optional)
- `wgpu` - GPU acceleration (optional)

### Serialization

- `serde` - JSON serialization
- `serde_json` - JSON
- `bincode` - Binary serialization
- `rkyv` - Zero-copy serialization (optional)

### Math and Numerics

- `blake3` - Hashing
- `ordered-float` - Ordered floating point
- `roaring` - Compressed bitmaps (optional)
- `petgraph` - Graph algorithms (optional)

## Features

### Default
Minimal governance-only features (no external crate deps)

### Full Feature Set
- **tiles** - Cognitum gate kernel integration
- **sona** - Sona adaptive thresholds
- **learned-rho** - GNN learned restrictions
- **hyperbolic** - Hyperbolic HNSW integration
- **mincut** - Subpolynomial partitioning
- **neural-gate** - Nervous system gating
- **attention** - Topology-gated attention
- **distributed** - Raft consensus
- **postgres** - PostgreSQL governance storage
- **simd** - SIMD optimizations
- **parallel** - Multi-threaded processing
- **spectral** - Spectral analysis
- **graph-integration** - Graph database integration
- **archive** - Rkyv serialization
- **gpu** - GPU acceleration
- **ruvllm** - LLM integration

### SIMD Variants
- **simd-avx2** - AVX2 SIMD
- **simd-avx512** - AVX-512 SIMD
- **simd-neon** - ARM NEON SIMD

## Testing

### Test Suites

1. **Integration Tests** (`tests/integration/`)
   - Full coherence engine tests
   - Integration with all modules

2. **Property Tests** (`tests/property/`)
   - Property-based testing
   - Invariant verification

3. **Replay Determinism** (`tests/replay_determinism.rs`)
   - Deterministic computation verification
   - Reproducibility tests

4. **Chaos Tests** (`tests/chaos_tests.rs`)
   - Failure injection
   - Recovery testing

5. **RuvLLM Integration** (`tests/ruvllm_integration_tests.rs`)
   - LLM coherence validation
   - Hallucination detection

6. **Storage Tests** (`tests/storage_tests.rs`)
   - Persistence verification
   - Recovery testing

## Benchmarks

### Comprehensive Benchmarks

1. **residual_bench.rs** - Residual computation
2. **energy_bench.rs** - Energy functionals
3. **gate_bench.rs** - Coherence gates
4. **incremental_bench.rs** - Incremental updates
5. **tile_bench.rs** - Tile operations
6. **sona_bench.rs** - Sona thresholds
7. **mincut_bench.rs** - Partitioning
8. **hyperbolic_bench.rs** - Hyperbolic operations
9. **coherence_bench.rs** - Coherence computation
10. **attention_bench.rs** - Attention gating
11. **coherence_benchmarks.rs** - Comprehensive coherence tests
12. **simd_benchmarks.rs** - SIMD performance
13. **gpu_benchmarks.rs** - GPU acceleration

## Examples

1. **basic_coherence.rs** - Basic coherence computation
2. **llm_validation.rs** - LLM output validation
3. **memory_tracking.rs** - Memory coherence tracking
4. **compute_ladder.rs** - Compute ladder optimization
5. **governance_audit.rs** - Governance system audit

## Usage Examples

```rust
use prime_radiant::{CoherenceEngine, SafetyVerifier};

// Create coherence engine
let engine = CoherenceEngine::new()?;

// Compute coherence score
let score = engine.compute_coherence(data)?;

// Verify LLM output for hallucinations
let verifier = SafetyVerifier::new();
let result = verifier.verify_output(llm_output, context)?;

if !result.is_consistent {
    println!("Potential hallucination detected!");
    println!("Inconsistencies: {:?}", result.violations);
}

// Distributed governance
if cfg!(feature = "distributed") {
    let governance = engine.distributed_governance()?;
    governance.propose_remediation(result.violations)?;
}
```

## Coherence Theory

### Sheaf Laplacian

The core mathematical framework uses sheaf theory to define coherence:

```
L = d* ∘ d + ρ
```

Where:
- `d` is the coboundary operator (restriction maps)
- `d*` is the adjoint (codifferential)
- `ρ` is the learned restriction (from GNN, mincut, etc.)

### Coherence Metrics

1. **Residual**
   - ||Lx - b||²
   - Measures inconsistency

2. **Energy**
   - xᵀLx
   - Measures coherence cost

3. **Gate**
   - H(x) where H is heaviside step
   - Binary coherence decision

### Integration with ADR-014

This module implements ADR-014 (Full Ecosystem Integration):
- Cognitum gate tiles for local coherence
- Sona for adaptive thresholds
- GNN for learned restrictions
- Mincut for hierarchical decomposition
- Hyperbolic HNSW for Poincare energy
- Nervous system for neural gating
- Attention for topology-aware computation
- Raft for distributed governance

## Use Cases

1. **AI Safety**
   - Hallucination detection
   - Consistency verification
   - Output validation

2. **Distributed Systems**
   - State consistency
   - Convergence verification
   - Governance protocols

3. **LLM Applications**
   - RAG coherence
   - Multi-agent consistency
   - Memory verification

4. **Knowledge Graphs**
   - Fact consistency
   - Relationship verification
   - Ontology validation

## Related Modules

- [cognitum-gate-kernel](../cognitum-gate-kernel/CLAUDE.md) - Cognitive tiles
- [sona](../sona/CLAUDE.md) - Adaptive thresholds
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Learned restrictions
- [ruvector-mincut](../ruvector-mincut/CLAUDE.md) - Graph partitioning
- [ruvector-hyperbolic-hnsw](../ruvector-hyperbolic-hnsw/CLAUDE.md) - Hyperbolic operations
- [ruvector-nervous-system](../ruvector-nervous-system/CLAUDE.md) - Neural gating
- [ruvector-attention](../ruvector-attention/CLAUDE.md) - Attention mechanisms
- [ruvllm](../ruvllm/CLAUDE.md) - LLM serving

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented universal coherence engine
- Identified sheaf Laplacian mathematics
- Catalogued ADR-014 ecosystem integration
- Listed 13 benchmark suites and 5 examples
