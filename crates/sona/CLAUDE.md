[根目录](../../CLAUDE.md) > **sona**

# sona - Module Documentation

## Module Responsibilities

Self-Optimizing Neural Architecture - runtime-adaptive learning for LLM routers and AI systems. Learns from feedback in sub-millisecond time without expensive retraining.

## Entry Points

- **`src/lib.rs`** - Main library entry point
- **`src/lora/`** - Two-tier LoRA implementation
- **`src/ewc/`** - Elastic Weight Consolidation++
- **`src/reasoning/`** - ReasoningBank storage
- **`src/adapters/`** - Platform adapters (WASM, NAPI)

## Key Interfaces

### Core Types
```rust
pub struct SonaModel {
    // Two-tier LoRA with fast/slow adapters
    fast_adapter: LoRAAdapter,  // <1ms updates
    slow_adapter: LoRAAdapter,  // Consolidated learning
}

pub struct EWC {
    // Fisher information matrix
    fisher: Array2<f32>,
    // Parameter importance
    importance: Array1<f32>,
}
```

### Learning API
```rust
impl SonaModel {
    pub fn forward(&self, input: &[f32]) -> Result<Vec<f32>>;
    pub fn update(&mut self, input: &[f32], feedback: &Feedback) -> Result<UpdateStats>;
    pub fn consolidate(&mut self) -> Result<ConsolidationStats>;
}
```

## Dependencies

### Required
- `parking_lot` - Fast synchronization
- `crossbeam` - Concurrent data structures
- `rand` - Random number generation

### Optional
- `serde` - Serialization support
- `wasm-bindgen` - WASM support
- `napi` - Node.js support

## Data Models

### Feedback Signal
```rust
pub struct Feedback {
    pub reward: f32,
    pub latency_ms: u64,
    pub metadata: HashMap<String, Value>,
}
```

### Trajectory
```rust
pub struct Trajectory {
    pub inputs: Vec<Vec<f32>>,
    pub outputs: Vec<Vec<f32>>,
    pub feedback: Vec<Feedback>,
    pub timestamp: DateTime<Utc>,
}
```

## Testing

### Running Tests
```bash
# All tests
cargo test -p ruvector-sona

# With features
cargo test -p ruvector-sona --features wasm

# Benchmarks
cargo bench -p ruvector-sona
```

## Key Files

| File | Description |
|------|-------------|
| `src/lib.rs` | Main exports |
| `src/lora/` | Two-tier LoRA implementation |
| `src/ewc/` | EWC++ for forgetting prevention |
| `src/reasoning/` | ReasoningBank |
| `src/adapters/wasm.rs` | WASM adapter |
| `src/adapters/napi.rs` | Node.js adapter |

## Performance Characteristics

### Adaptation Speed
- Fast adapter update: <1ms
- Slow adapter consolidation: 100-500ms
- Full consolidation: 1-5s

### Memory
- Fast adapter: ~1-10MB
- Slow adapter: ~10-100MB
- ReasoningBank: Variable

## Feature Flags

- `default` - serde support
- `wasm` - WebAssembly support
- `napi` - Node.js bindings
- `serde-support` - Serialization

## Common Patterns

### Basic Usage
```rust
use sona::{SonaModel, SonaConfig};

let config = SonaConfig {
    input_dim: 768,
    output_dim: 10,
    fast_lr: 0.01,
    slow_lr: 0.001,
    ..Default::default()
};

let mut model = SonaModel::new(config)?;

// Forward pass
let output = model.forward(&input)?;

// Update from feedback
let feedback = Feedback { reward: 0.8, ..Default::default() };
let stats = model.update(&input, &feedback)?;
```

### Consolidation
```rust
// Periodic consolidation
let stats = model.consolidate()?;
```

### WASM Usage
```javascript
import { SonaModel } from 'sona-wasm';

const model = new SonaModel(config);
const output = model.forward(input);
model.update(input, feedback);
```

## Related Modules

- **[ruvector-router-core](../ruvector-router-core/)** - Semantic routing
- **[ruvector-core](../ruvector-core/)** - Vector database
- **[ruvector-postgres](../ruvector-postgres/)** - PostgreSQL integration

## Changelog

### 2026-03-09
- Initial module documentation generated
- Documented two-tier LoRA architecture
- Identified EWC++ integration
