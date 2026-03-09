# ruvector-wasm

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-wasm**

## Module Description

WebAssembly bindings for Ruvector including kernel pack system (ADR-005), providing browser-compatible vector database operations with 58KB optimized bundle size. This module enables high-performance vector operations in web applications without server dependencies.

## Entry Points

- **Main Library**: `src/lib.rs` - WASM bindings and kernel pack system

## Architecture

### Core Components

1. **Vector Operations**
   - In-memory vector storage
   - Distance calculations
   - Similarity search
   - Batch operations

2. **Kernel Pack System (ADR-005)**
   - Sandboxed compute kernels
   - Kernel verification
   - Signing capability
   - Cryptographic security

3. **IndexedDB Integration**
   - Persistent storage
   - Browser-native database
   - Transaction support
   - Async operations

4. **Collections** (optional, not available in WASM)
   - File I/O dependent features
   - Not available in browser
   - Provided for API compatibility

## Key Dependencies

- `ruvector-core` - Core vector operations (memory-only, uuid-support)
- `ruvector-collections` - Collections (optional, not functional in WASM)
- `ruvector-filter` - Filter (optional, not functional in WASM)
- `parking_lot` - Fast mutexes
- `getrandom` - Random number generation
- `wasm-bindgen` - WASM bindings
- `wasm-bindgen-futures` - Async WASM
- `js-sys` - JavaScript system bindings
- `web-sys` - Web API bindings
- `thiserror` - Error handling
- `anyhow` - Error handling
- `serde` - Serialization
- `serde_json` - JSON serialization
- `serde-wasm-bindgen` - Serde WASM bridge

### Cryptography Dependencies (Kernel Pack)

- `sha2` - SHA-256 hashing (optional)
- `ed25519-dalek` - Ed25519 signatures (optional)
- `hex` - Hex encoding (optional)
- `base64` - Base64 encoding (optional)
- `rand` - Random signing (optional)

## Features

- **default** - Basic WASM functionality
- **simd** - Enable SIMD optimizations
- **collections** - Collections API (non-functional in WASM)
- **kernel-pack** - Enable kernel pack system (ADR-005)
- **signing** - Enable kernel signing capability

## Web API Features

```rust
web-sys features = [
    "console",
    "Window",
    "IdbDatabase",
    "IdbFactory",
    "IdbObjectStore",
    "IdbRequest",
    "IdbTransaction",
    "IdbOpenDbRequest",
]
```

## Optimization Profile

```toml
[profile.release]
opt-level = "z"      # Optimize for size
lto = true           # Link-time optimization
codegen-units = 1    # Single codegen unit
panic = "abort"      # Abort on panic

[profile.release.package."*"]
opt-level = "z"      # Optimize all packages for size
```

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **WASM Tests**: Use `wasm-bindgen-test` for browser testing

## Usage Examples

```javascript
import { VectorDB, KernelPack } from 'ruvector-wasm';

// Create vector database
const db = new VectorDB();

// Insert vectors
db.insert('doc-1', new Float32Array([0.1, 0.2, 0.3]));

// Search similar vectors
const results = db.search(new Float32Array([0.15, 0.25, 0.35]), 10);

// Use IndexedDB for persistence
await db.persist('my-vector-db');

// Kernel pack system (ADR-005)
const kernel = new KernelPack(kernelCode);
await kernel.verify(signature);
const result = kernel.execute(input);

// Sign kernels
const signed = KernelPack.sign(kernelCode, privateKey);
```

## Kernel Pack System (ADR-005)

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Kernel Pack (ADR-005)                  │
│                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Kernel    │  │  Signature  │  │  Metadata   │     │
│  │     Code    │  │  (Ed25519)  │  │  (JSON)     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │                 │                 │            │
│         └─────────────────┴─────────────────┘            │
│                           │                              │
│                      ┌────▼────┐                         │
│                      │  Pack   │                         │
│                      └─────────┘                         │
│                                                           │
│  Verification: SHA-256(code) + Ed25519 verify           │
│  Execution: Sandboxed WASM runtime                       │
└─────────────────────────────────────────────────────────┘
```

### Benefits

1. **Security** - Cryptographic verification
2. **Sandboxing** - Isolated execution
3. **Portability** - Cross-platform kernels
4. **Versioning** - Metadata tracking
5. **Signing** - Author verification

## Browser Compatibility

- **Chrome**: 57+
- **Firefox**: 52+
- **Safari**: 11+
- **Edge**: 16+

## Bundle Size

- **Base**: ~58KB (gzipped)
- **With SIMD**: ~62KB (gzipped)
- **With Kernel Pack**: ~65KB (gzipped)

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-node](../ruvector-node/CLAUDE.md) - Node.js bindings
- [ruvector-gnn-wasm](../ruvector-gnn-wasm/CLAUDE.md) - GNN WASM
- [ruvector-graph-wasm](../ruvector-graph-wasm/CLAUDE.md) - Graph WASM

## Use Cases

1. **Client-Side Search** - Browser-based vector search
2. **Offline Applications** - PWA with local vector DB
3. **Privacy** - No server required
4. **Edge Computing** - Computation at the edge
5. **Interactive Demos** - Real-time demonstrations

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented WASM bindings and kernel pack system
- Identified optimization features and bundle size
- Catalogued browser compatibility and use cases
