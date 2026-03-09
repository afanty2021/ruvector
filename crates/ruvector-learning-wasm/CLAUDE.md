# ruvector-learning-wasm

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-learning-wasm**

## Module Description

Ultra-fast MicroLoRA adaptation for WebAssembly, providing rank-2 LoRA (Low-Rank Adaptation) with <100μs latency for per-operator learning in browser environments. This module enables on-device fine-tuning of neural networks without sending data to servers.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API for MicroLoRA WASM

## Architecture

### Core Components

1. **MicroLoRA Engine**
   - Rank-2 decomposition
   - Ultra-low latency adaptation
   - Browser-optimized computation
   - Minimal memory footprint

2. **Per-Operator Learning**
   - Layer-wise adaptation
   - Independent module training
   - Fast convergence
   - Gradient-free optimization

3. **WASM Optimizations**
   - Size optimization (opt-level = "z")
   - LTO enabled
   - Single codegen unit
   - Panic = abort

## Key Dependencies

- `wasm-bindgen` - WASM bindings
- `js-sys` - JavaScript system bindings
- `serde` - Serialization (optional)
- `serde-wasm-bindgen` - Serde WASM bridge (optional)

## Features

- **default** - Standard library support
- **std** - Standard library (enabled by default)
- **serde** - Serialization support
- **simd** - SIMD optimizations (when available)

## Performance Characteristics

- **Latency**: <100μs per adaptation
- **Rank**: Fixed at rank-2 for minimal overhead
- **Size**: Optimized for smallest WASM binary
- **Memory**: Minimal memory footprint

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **WASM Tests**: Use `wasm-bindgen-test` for browser testing

## Usage Examples

```javascript
import { MicroLoRA } from 'ruvector-learning-wasm';

// Create MicroLoRA adapter
const adapter = new MicroLoRA(inputDim, outputDim);

// Adapt with new data
const delta = adapter.adapt(newData, learningRate);

// Apply to base model
const adapted = adapter.applyDelta(baseModel, delta);

// Use adapted model
const prediction = adapted.forward(input);
```

## MicroLoRA Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     MicroLoRA (rank-2)                   │
│                                                           │
│  Input ──► [Linear] ──► [LoRA A] ──► [LoRA B] ──► Add   │
│                ▲               │           │             │
│                │               ▼           ▼             │
│            ┌─────┐         [A ∈ R²ˣᵈ]  [B ∈ Rᵈˣ²]       │
│            │  Δ  │                                     │
│            └─────┘                                     │
│                                                          │
│  Parameters: 2×d + d×2 = 4d (vs d×d for full layer)    │
│  Speedup: ~d/4x for large d                            │
└─────────────────────────────────────────────────────────┘
```

## Optimization Features

1. **Rank-2 Decomposition**
   - Minimal parameter overhead
   - Fast computation
   - Good approximation for small changes

2. **Browser Optimized**
   - No WebGPU required
   - Pure WASM computation
   - Works on all browsers

3. **Privacy Preserving**
   - No data leaves browser
   - On-device training
   - Local inference

## Use Cases

1. **Personalization** - Adapt models to user preferences
2. **Fine-tuning** - Specialize models for specific tasks
3. **Privacy** - Train without sending data to servers
4. **Edge AI** - Learning on resource-constrained devices
5. **Interactive ML** - Real-time model adaptation

## Related Modules

- [ruvector-gnn-wasm](../ruvector-gnn-wasm/CLAUDE.md) - GNN WASM bindings
- [ruvector-wasm](../ruvector-wasm/CLAUDE.md) - Core WASM bindings
- [ruvector-attention-wasm](../ruvector-attention-wasm/CLAUDE.md) - Attention WASM

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented MicroLoRA architecture
- Identified performance characteristics
- Catalogued optimization features and use cases
