# ruvector-node

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-node**

## Module Description

Node.js bindings for Ruvector via NAPI-RS, providing high-performance vector operations, collections, filtering, and metrics to JavaScript/TypeScript applications. This module enables native-speed vector database operations in Node.js.

## Entry Points

- **Main Library**: `src/lib.rs` - NAPI-RS bindings
- **Build**: `build.rs` - NAPI build configuration

## Architecture

### Core Bindings

1. **Vector Operations**
   - Vector creation and manipulation
   - Distance calculations
   - Similarity search
   - Batch operations

2. **Collections**
   - Vector collections
   - Index management
   - CRUD operations
   - Query execution

3. **Filter**
   - Metadata filtering
   - Predicate evaluation
   - Compound filters
   - Filter optimization

4. **Metrics**
   - Performance metrics
   - Query statistics
   - Resource usage
   - Custom metrics

## Key Dependencies

- `ruvector-core` - Core vector operations
- `ruvector-collections` - Collection operations
- `ruvector-filter` - Filtering operations
- `ruvector-metrics` - Metrics collection
- `napi` - NAPI-RS framework
- `napi-derive` - NAPI derive macros
- `tokio` - Async runtime
- `thiserror` - Error handling
- `anyhow` - Error handling
- `tracing` - Logging
- `serde` - Serialization
- `serde_json` - JSON serialization

## Features

All features are enabled by default for maximum Node.js functionality.

## Build Configuration

```toml
[lib]
crate-type = ["cdylib"]

[profile.release]
lto = true
strip = true
```

## NAPI Build

```rust
// build.rs
extern crate napi_build;

fn main() {
    napi_build::setup();
}
```

## Platform Support

- **macOS**: x64, arm64
- **Linux**: x64, arm64 (glibc)
- **Windows**: x64 (MSVC)

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Integration Tests**: Test Node.js integration

## Usage Examples

```javascript
const { VectorDB, DistanceMetric } = require('ruvector-node');

// Create vector database
const db = new VectorDB();

// Insert vectors
db.insert('doc-1', [0.1, 0.2, 0.3], { category: 'tech' });

// Search similar vectors
const results = db.search(
    [0.15, 0.25, 0.35],
    DistanceMetric.Cosine,
    10  // top-k
);

// Apply filters
const filtered = db.searchWithFilter(
    query,
    DistanceMetric.Euclidean,
    10,
    { category: 'tech' }
);

// Get metrics
const metrics = db.getMetrics();
console.log(`Queries: ${metrics.queryCount}`);
```

## Async Operations

```javascript
// Async operations with tokio
const results = await db.searchAsync(query, k);
const inserted = await db.insertAsync(id, vector, metadata);
```

## Error Handling

```javascript
try {
    db.insert(id, vector);
} catch (error) {
    console.error('Insert failed:', error.message);
}
```

## Related Modules

- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-collections](../ruvector-collections/CLAUDE.md) - Collections
- [ruvector-filter](../ruvector-filter/CLAUDE.md) - Filtering
- [ruvector-metrics](../ruvector-metrics/CLAUDE.md) - Metrics
- [ruvector-wasm](../ruvector-wasm/CLAUDE.md) - WASM bindings

## NPM Package

This module is published to npm as `@ruvector/core`.

## Performance Characteristics

- **Zero-Copy**: Minimize data copying between Rust and JS
- **Async**: Non-blocking operations with Tokio
- **Parallel**: Multi-core processing with Rayon
- **SIMD**: Hardware acceleration where available

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented NAPI-RS bindings architecture
- Identified platform support and features
- Catalogued usage examples and performance
