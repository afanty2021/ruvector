# ruqu-core

[根目录](../../CLAUDE.md) > [crates](../) > **ruqu-core**

## Module Description

High-performance quantum circuit simulator in pure Rust, providing state-vector simulation with SIMD acceleration, noise models, and multi-threading for quantum computing research and education.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and quantum simulator
- **State Vector**: Quantum state representation
- **Gates**: Quantum gate operations
- **Noise**: Noise models and simulation

## Architecture

### Core Components

1. **State Vector**
   - Complex amplitude storage
   - State manipulation
   - Measurement operations
   - State initialization

2. **Quantum Gates**
   - Single-qubit gates
   - Multi-qubit gates
   - Parameterized gates
   - Custom gates

3. **Circuit**
   - Gate composition
   - Circuit optimization
   - Parallel execution
   - Circuit visualization

4. **Noise Models**
   - Depolarizing noise
   - Amplitude damping
   - Phase damping
   - Custom noise channels

## Key Dependencies

- `rand` - Random number generation
- `thiserror` - Error handling
- `serde` - Serialization (optional)
- `rayon` - Parallel processing (optional)
- `tracing` - Logging (optional)

## Features

- **default** - Standard library support
- **std** - Standard library (enabled by default)
- **parallel** - Multi-threaded execution with Rayon
- **simd** - SIMD optimizations

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Benchmarks**:
  - `benches/quantum_sim.rs` - Quantum simulation benchmarks

## Usage Examples

```rust
use ruqu_core::{QuantumState, Gate, Circuit};

// Create quantum state
let mut state = QuantumState::new(2)?;  // 2 qubits

// Apply gates
state.apply_gate(Gate::Hadamard, 0)?;  // H on qubit 0
state.apply_gate(Gate::CNOT, [0, 1])?;  // CNOT(0, 1)

// Measure
let result = state.measure_all()?;

// Build circuit
let mut circuit = Circuit::new(2);
circuit.add_gate(Gate::Hadamard, 0);
circuit.add_gate(Gate::PauliX, 1);
circuit.add_gate(Gate::CNOT, [0, 1]);

// Execute circuit
let result = circuit.execute()?;

// Add noise
let noisy_result = circuit.with_noise(NoiseModel::depolarizing(0.01))?;
```

## Quantum Gates

### Single-Qubit Gates

- **Hadamard (H)** - Superposition
- **Pauli X** - Bit flip
- **Pauli Y** - Bit and phase flip
- **Pauli Z** - Phase flip
- **S** - Phase gate (π/2)
- **T** - π/8 gate
- **RX(θ)** - Rotation around X
- **RY(θ)** - Rotation around Y
- **RZ(θ)** - Rotation around Z

### Multi-Qubit Gates

- **CNOT** - Controlled-NOT
- **CZ** - Controlled-Z
- **SWAP** - Swap qubits
- **Toffoli** - Controlled-CNOT
- **Fredkin** - Controlled-SWAP

## Noise Models

### Depolarizing Noise

```rust
let noise = NoiseModel::depolarizing(0.01);  // 1% error rate
```

### Amplitude Damping

```rust
let noise = NoiseModel::amplitude_damping(0.1);
```

### Phase Damping

```rust
let noise = NoiseModel::phase_damping(0.05);
```

### Combined Noise

```rust
let noise = NoiseModel::combined()
    .with_depolarizing(0.01)
    .with_amplitude_damping(0.05)
    .with_phase_damping(0.02);
```

## Performance Characteristics

- **State Storage**: O(2ⁿ) where n = number of qubits
- **Gate Application**: O(2ⁿ) for general gates
- **Measurement**: O(1) for single qubit, O(n) for all qubits
- **Parallel**: Linear speedup with number of cores
- **SIMD**: 2-4x speedup for gate operations

## Use Cases

1. **Quantum Algorithm Research**
   - Test quantum algorithms
   - Benchmark performance
   - Validate theoretical results

2. **Education**
   - Learn quantum computing
   - Visualize circuits
   - Understand noise

3. **Application Development**
   - Prototype quantum algorithms
   - Test error correction
   - Develop hybrid algorithms

4. **Noise Simulation**
   - Study noise effects
   - Test error correction
   - Optimize circuits

## Related Modules

- [ruqu-algorithms](../ruqu-algorithms/CLAUDE.md) - Quantum algorithms
- [ruqu-exotic](../ruqu-exotic/CLAUDE.md) - Exotic quantum models
- [ruqu-wasm](../ruqu-wasm/CLAUDE.md) - WASM bindings

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented quantum circuit simulator
- Identified gate operations and noise models
- Catalogued performance characteristics and use cases
