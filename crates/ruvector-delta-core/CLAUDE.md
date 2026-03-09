# ruvector-delta-core

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-delta-core**

## Module Description

Core delta types and traits for behavioral vector change tracking, enabling incremental updates, streaming operations, and efficient state synchronization in vector databases and machine learning systems.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and delta core
- **Delta Operations**: `src/delta.rs` - Delta type definitions
- **Encoding**: `src/encoding.rs` - Efficient delta encoding
- **Compression**: `src/compression.rs` - Compression support
- **Streaming**: `src/stream.rs` - Streaming delta operations
- **Windowing**: `src/window.rs` - Time-based windowing

## Architecture

### Core Components

1. **Delta Types** (`delta.rs`)
   - `Delta<T>` - Encapsulates state changes
   - `DeltaType` - Type of change (Insert, Update, Delete)
   - `DeltaBatch<T>` - Batched operations
   - Delta composition and reduction

2. **Encoding** (`encoding.rs`)
   - Compact binary representation
   - Varint encoding for sizes
   - Delta compression
   - Zero-copy deserialization

3. **Compression** (`compression.rs`)
   - LZ4 compression (fast)
   - ZSTD compression (better ratio)
   - Compression level control
   - Adaptive compression

4. **Streaming** (`stream.rs`)
   - Delta streams
   - Backpressure handling
   - Batching strategies
   - Flow control

5. **Windowing** (`window.rs`)
   - Time-based windows
   - Count-based windows
   - Sliding windows
   - Tumbling windows

6. **Error Handling** (`error.rs`)
   - Error types
   - Error propagation
   - Recovery strategies

## Key Dependencies

- `thiserror` - Error handling
- `bincode` - Binary serialization
- `serde` - JSON serialization (optional)
- `serde_json` - JSON serialization (optional)
- `lz4_flex` - LZ4 compression (optional)
- `zstd` - ZSTD compression (optional)
- `parking_lot` - Fast mutexes
- `smallvec` - Small vector optimization
- `arrayvec` - Fixed-size arrays

## Features

- **std** - Standard library support (default)
- **simd** - SIMD optimizations (optional)
- **serde** - Serialization support (optional)
- **compression** - Compression support (optional)

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Property Tests**: Use `proptest` for property-based testing
- **Benchmarks**: Planned but not yet implemented

## Usage Examples

```rust
use ruvector_delta_core::{Delta, DeltaType, DeltaBatch};

// Create individual deltas
let insert = Delta::insert(vec![1.0, 2.0, 3.0]);
let update = Delta::update(vec![4.0, 5.0, 6.0]);
let delete = Delta::delete();

// Batch deltas
let mut batch = DeltaBatch::new();
batch.add(insert);
batch.add(update);
batch.add(delete);

// Compress batch
let compressed = batch.compress()?;

// Encode for transmission
let encoded = batch.encode()?;

// Decode and apply
let decoded = DeltaBatch::decode(&encoded)?;
decoded.apply_to(&mut state)?;
```

## Delta Type System

```rust
pub enum Delta<T> {
    Insert(T),
    Update(T),
    Delete,
    Batch(Vec<Delta<T>>),
}

pub struct DeltaBatch<T> {
    pub deltas: Vec<Delta<T>>,
    pub timestamp: DateTime<Utc>,
    pub compression: CompressionType,
}
```

## Performance Characteristics

- **Encoding**: O(n) where n = delta size
- **Compression**: O(n) with tradeoff between speed (LZ4) and ratio (ZSTD)
- **Batching**: O(1) amortized per delta
- **Windowing**: O(log w) where w = window size

## Use Cases

1. **Incremental Learning** - Update models without full retraining
2. **Streaming ML** - Process data in streaming fashion
3. **Real-time Updates** - Low-latency state propagation
4. **Distributed Sync** - Efficient state synchronization
5. **Change Tracking** - Audit logs and reproducibility

## Related Modules

- [ruvector-delta-index](../ruvector-delta-index/CLAUDE.md) - Delta-aware HNSW
- [ruvector-delta-graph](../ruvector-delta-graph/CLAUDE.md) - Graph deltas
- [ruvector-delta-consensus](../ruvector-delta-consensus/CLAUDE.md) - Delta consensus
- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented 6 core source files
- Identified delta type system and features
- Catalogued compression options and use cases
