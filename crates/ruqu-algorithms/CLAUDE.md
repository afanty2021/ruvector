# ruqu-algorithms

[根目录](../../CLAUDE.md) > [crates](../) > **ruqu-algorithms**

## Module Description

Production-ready quantum algorithms in Rust, implementing VQE for chemistry, Grover's search, QAOA optimization, and Surface Code error correction for practical quantum computing applications.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and algorithm implementations
- **VQE**: Variational Quantum Eigensolver
- **Grover**: Grover's search algorithm
- **QAOA**: Quantum Approximate Optimization Algorithm
- **Surface Code**: Quantum error correction

## Architecture

### Core Algorithms

1. **VQE (Variational Quantum Eigensolver)**
   - Molecular Hamiltonians
   - Ansatz circuits
   - Classical optimization
   - Chemistry applications

2. **Grover's Search**
   - Oracle construction
   - Diffusion operator
   - Amplitude amplification
   - Database search

3. **QAOA (Quantum Approximate Optimization Algorithm)**
   - Combinatorial optimization
   - Mixer Hamiltonians
   - Cost Hamiltonians
   - Parameter optimization

4. **Surface Code**
   - Lattice surgery
   - Syndrome measurement
   - Error decoding
   - Fault tolerance

## Key Dependencies

- `ruqu-core` - Quantum circuit simulator
- `rand` - Random number generation
- `thiserror` - Error handling
- `serde` - Serialization (optional)
- `tracing` - Logging (optional)

## Features

- **default** - Standard library support
- **std** - Standard library (enabled by default)

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Benchmarks**: Use `criterion` for performance benchmarks

## Usage Examples

### VQE for Molecular Ground State

```rust
use ruqu_algorithms::{vqe::VQE, vqe::molecular_hamiltonian};

// Define molecule (H2)
let hamiltonian = molecular_hamiltonian("H2")?;

// Create ansatz
let ansatz = Ansatz::hardware_efficient(num_qubits, depth);

// Run VQE
let mut vqe = VQE::new(hamiltonian, ansatz);
let (energy, params) = vqe.run()?;  // Find ground state energy
```

### Grover's Search

```rust
use ruqu_algorithms::grover::{Grover, Oracle};

// Create oracle for searching database
let oracle = Oracle::from_database(|item| item == target);

// Run Grover's search
let mut grover = Grover::new(oracle, num_items)?;
let result = grover.search()?;  // Find target in O(√N)

// Probability of success
println!("Found with probability: {}", result.probability);
```

### QAOA for MaxCut

```rust
use ruqu_algorithms::qaoa::{QAOA, MaxCut};

// Define MaxCut problem
let problem = MaxCut::new(graph)?;

// Create QAOA with p layers
let mut qaoa = QAOA::new(problem, depth=p)?;

// Optimize parameters
let solution = qaoa.optimize()?;  // Find approximate solution
```

### Surface Code Error Correction

```rust
use ruqu_algorithms::surface_code::{SurfaceCode, Decoder};

// Create surface code lattice
let code = SurfaceCode::new(d=5)?;  // Distance-5 code

// Simulate errors
let syndromes = code.simulate_errors(error_rate)?;

// Decode using minimum-weight perfect matching
let decoder = Decoder::mwpm();
let corrections = decoder.decode(&syndromes)?;

// Apply corrections
code.apply_corrections(corrections)?;
```

## Algorithm Details

### VQE (Variational Quantum Eigensolver)

**Applications:**
- Molecular ground states
- Material properties
- Quantum chemistry
- Drug discovery

**Ansatz Types:**
- Hardware-efficient
- Unitary Coupled Cluster (UCCSD)
- Quantum Approximate Optimization (QAOA)
- Problem-inspired

**Optimizers:**
- COBYLA
- SPSA
- Gradient descent
- Natural gradient

### Grover's Search

**Complexity:**
- Query: O(√N) vs O(N) classical
- Space: O(log N) qubits
- Success probability: 1 - O(1/N)

**Variants:**
- Fixed-point Grover
- Amplitude amplification
- Quantum counting
- Database search

### QAOA (Quantum Approximate Optimization Algorithm)

**Applications:**
- MaxCut
- Max-2SAT
- TSP (Traveling Salesman)
- Portfolio optimization

**Parameters:**
- Depth (p): Circuit depth
- β, γ: Rotation angles
- Mixer: X-mixer or XY-mixer
- Cost: Problem Hamiltonian

### Surface Code

**Metrics:**
- Distance (d): Number of errors corrected
- Threshold: ~1% error rate
- Overhead: ~d² physical qubits per logical qubit
- Lattice: Rotated or standard

**Decoding:**
- Minimum-weight perfect matching
- Union-find
- Neural network
- Tensor network

## Performance Characteristics

### VQE
- **Circuit Depth**: O(n) for chemistry
- **Measurements**: O(1/ε²) per iteration
- **Iterations**: O(poly(n)) for convergence
- **Accuracy**: Chemical accuracy (1 mHa)

### Grover
- **Queries**: O(√N) to find target
- **Success Probability**: 1 - O(1/N)
- **Circuit Depth**: O(√N) with diffusion
- **Qubits**: O(log N) for database

### QAOA
- **Circuit Depth**: O(p·n) where p = depth
- **Quality**: Approximation ratio improves with p
- **Optimization**: Classical parameter optimization
- **Qubits**: O(n) for problem size n

### Surface Code
- **Threshold**: ~1% physical error rate
- **Overhead**: O(d²) physical per logical qubit
- **Decoding Time**: O(d²) for MWPM
- **Latency**: O(d) cycles for correction

## Use Cases

1. **Quantum Chemistry**
   - Molecular simulation
   - Ground state energy
   - Reaction dynamics

2. **Optimization**
   - Combinatorial optimization
   - Scheduling
   - Resource allocation

3. **Search**
   - Database search
   - Unstructured search
   - Amplitude amplification

4. **Error Correction**
   - Fault-tolerant quantum computing
   - Threshold optimization
   - Decoding algorithms

## Related Modules

- [ruqu-core](../ruqu-core/CLAUDE.md) - Quantum simulator
- [ruqu-exotic](../ruqu-exotic/CLAUDE.md) - Exotic models
- [ruqu-wasm](../ruqu-wasm/CLAUDE.md) - WASM bindings

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented production-ready quantum algorithms
- Identified VQE, Grover, QAOA, and Surface Code
- Catalogued algorithm details and use cases
