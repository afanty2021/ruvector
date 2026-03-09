# ruvector-solver

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-solver**

## Module Description

Sublinear-time solver library providing O(log n) to O(√n) algorithms for sparse linear systems, PageRank, spectral methods, and conjugate gradient optimization. This module enables ultra-fast approximations for massive-scale problems where exact computation is infeasible.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and solver orchestration
- **Neumann Solver**: `src/neumann.rs` - Neumann series solver
- **Conjugate Gradient**: `src/cg.rs` - CG solver
- **Forward/Backward Push**: `src/forward_push.rs`, `src/backward_push.rs` - Push-based solvers
- **Random Walk**: `src/random_walk.rs` - Random walk solver
- **True Solver**: `src/true_solver.rs` - Exact solver implementation

## Architecture

### Core Solvers

1. **Neumann Series** (`neumann.rs`)
   - Approximate matrix inversion
   - Suitable for well-conditioned systems
   - O(log n) convergence

2. **Conjugate Gradient** (`cg.rs`)
   - Standard CG algorithm
   - For symmetric positive-definite systems
   - Fast convergence with good preconditioning

3. **Forward Push** (`forward_push.rs`)
   - Personalized PageRank
   - Localized computation
   - O(1/ε) iterations for ε-approximation

4. **Backward Push** (`backward_push.rs`)
   - Reverse personalized PageRank
   - Complementary to forward push
   - Efficient for reverse queries

5. **Hybrid Random Walk** (`random_walk.rs`)
   - Monte Carlo-based solving
   - Embarrassingly parallel
   - Good for very large sparse systems

6. **True Solver** (`true_solver.rs`)
   - Exact solver (slower but accurate)
   - Based on Neumann with full convergence
   - Ground truth for validation

7. **BMSSP** (`bmssp.rs`)
   - Batch Multi-Source Shortest Paths
   - Batch distance computation
   - Optimized for multiple queries

### Infrastructure

1. **Budget Management** (`budget.rs`)
   - Computational budget tracking
   - Adaptive precision control
   - Resource allocation

2. **Arena Allocation** (`arena.rs`)
   - Memory pool management
   - Efficient allocation/deallocation
   - Reduces fragmentation

3. **Validation** (`validation.rs`)
   - Solution verification
   - Error bounds checking
   - Quality metrics

4. **Events** (`events.rs`)
   - Event-driven computation
   - Progress tracking
   - Callback system

5. **Router** (`router.rs`)
   - Problem type detection
   - Solver selection
   - Adaptive routing

6. **Audit** (`audit.rs`)
   - Computation auditing
   - Reproducibility guarantees
   - Debugging support

7. **SIMD** (`simd.rs`)
   - SIMD-accelerated operations
   - Vectorized computation
   - Platform-specific optimizations

### Types and Traits

1. **Types** (`types.rs`)
   - Problem formulations
   - Solution types
   - Error types

2. **Traits** (`traits.rs`)
   - Solver trait definitions
   - Matrix interfaces
   - Vector interfaces

3. **Error** (`error.rs`)
   - Error types
   - Error handling
   - Diagnostics

## Key Dependencies

- `nalgebra` - Linear algebra (optional, nalgebra-backend feature)
- `rayon` - Parallel processing (optional, parallel feature)
- `crossbeam` - Concurrent data structures (optional, parallel feature)
- `rand` - Random number generation
- `dashmap` - Concurrent hashmap
- `parking_lot` - Fast mutexes

## Features

- **neumann** - Neumann series solver
- **cg** - Conjugate gradient solver
- **forward-push** - Forward push solver (Personalized PageRank)
- **backward-push** - Backward push solver
- **hybrid-random-walk** - Monte Carlo random walk solver
- **true-solver** - Exact solver (Neumann-based)
- **bmssp** - Batch Multi-Source Shortest Paths
- **nalgebra-backend** - Use nalgebra for linear algebra
- **parallel** - Enable parallel processing
- **simd** - Enable SIMD optimizations
- **wasm** - WASM compatibility
- **full** - All algorithms and features
- **all-algorithms** - All solver algorithms

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Benchmarks**:
  - `benches/solver_baseline.rs` - Baseline performance
  - `benches/solver_neumann.rs` - Neumann solver benchmarks
  - `benches/solver_cg.rs` - CG solver benchmarks
  - `benches/solver_push.rs` - Push solver benchmarks
  - `benches/solver_e2e.rs` - End-to-end benchmarks

## Platform Bindings

- **Node.js**: `ruvector-solver-node` - NAPI-RS bindings
- **WASM**: `ruvector-solver-wasm` - WebAssembly bindings

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph algorithms
- [ruvector-mincut](../ruvector-mincut/CLAUDE.md) - Graph partitioning

## Usage Examples

```rust
use ruvector_solver::{Solver, SolverType};

// Create a solver
let solver = Solver::new(SolverType::ForwardPush);

// Set up a problem (e.g., PageRank)
let problem = setup_pagerank_problem()?;

// Solve with computational budget
let solution = solver.solve_with_budget(problem, 1e-6)?;

// Verify solution quality
let error = solution.verify()?;
```

## Performance Characteristics

- **Neumann**: O(log n) iterations, O(nnz) per iteration
- **Forward/Backward Push**: O(1/ε) iterations, O(degree) per iteration
- **CG**: O(√κ) iterations (κ = condition number)
- **Random Walk**: O(√n) samples for ε-approximation
- **BMSSP**: O(n log n + m) for batch processing

## Solver Selection Guide

| Problem Type | Recommended Solver | Feature |
|-------------|-------------------|---------|
| Well-conditioned linear system | Neumann | neumann |
| Symmetric positive-definite | CG | cg |
| Personalized PageRank | Forward Push | forward-push |
| Reverse PageRank | Backward Push | backward-push |
| Very large, sparse | Hybrid Random Walk | hybrid-random-walk |
| Ground truth needed | True Solver | true-solver |
| Multiple shortest paths | BMSSP | bmssp |

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 18 source files across solvers and infrastructure
- Identified 7 solver algorithms with complexity analysis
- Catalogued 5 benchmark suites
