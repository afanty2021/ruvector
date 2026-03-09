# ruvector-delta-index

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-delta-index**

## Module Description

Delta-aware HNSW index with incremental updates and repair strategies, enabling efficient dynamic vector indexing without full reconstruction. This module extends HNSW (Hierarchical Navigable Small World) to support fast insertions, deletions, and updates.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and delta-aware HNSW
- **Delta Integration**: Uses `ruvector-delta-core` for delta representation

## Architecture

### Core Components

1. **Delta-Aware HNSW**
   - Incremental graph construction
   - Lazy index updates
   - Efficient repair strategies
   - Background reindexing

2. **Insertion Optimization**
   - Bulk insert support
   - Candidate pruning
   - Neighbor selection optimization
   - Parallel construction

3. **Deletion Support**
   - Soft delete with tombstones
   - Lazy graph repair
   - Connectivity maintenance
   - Garbage collection

4. **Update Strategies**
   - In-place updates
   - Delete-insert optimization
   - Delta propagation
   - Consistency maintenance

## Key Dependencies

- `ruvector-delta-core` - Core delta types and traits
- `thiserror` - Error handling
- `parking_lot` - Fast mutexes
- `dashmap` - Concurrent hashmap
- `smallvec` - Small vector optimization
- `priority-queue` - Priority queue for HNSW
- `rand` - Random number generation
- `rand_xorshift` - XOR shift random generator
- `rayon` - Parallel processing (optional)
- `simsimd` - SIMD distance calculations (optional)
- `bincode` - Serialization (optional)
- `serde` - JSON serialization (optional)

## Features

- **default** - Enable parallel mode
- **parallel** - Enable parallel processing with Rayon
- **simd** - Enable SIMD optimizations for distance calculations
- **persistence** - Enable serialization and persistence

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **Property Tests**: Use `proptest` for property-based testing
- **Benchmarks**: Planned but not yet implemented

## Usage Examples

```rust
use ruvector_delta_index::{DeltaHnsw, Delta};
use ruvector_delta_core::DeltaType;

// Create delta-aware HNSW
let mut index = DeltaHnsw::new(dimensions, ef_construction)?;

// Apply delta insertions
for vector in vectors {
    let delta = Delta::insert(vector);
    index.apply_delta(delta)?;
}

// Apply delta deletions
let delta = Delta::delete(id);
index.apply_delta(delta)?;

// Force index repair
index.repair()?;

// Search with approximate nearest neighbor
let results = index.search(query, k)?;
```

## Delta-Aware Algorithms

1. **Incremental Construction**
   - Add new points without full rebuild
   - Update neighbor lists incrementally
   - Maintain graph properties

2. **Lazy Repair**
   - Defer repair until query time
   - Batch repairs for efficiency
   - Background repair threads

3. **Consistency Maintenance**
   - Preserve connectivity
   - Avoid isolated components
   - Maintain small-world properties

## Performance Characteristics

- **Insert**: O(log n) amortized (vs O(n) for rebuild)
- **Delete**: O(log n) with lazy repair
- **Update**: O(log n) with in-place optimization
- **Search**: O(log n) same as static HNSW
- **Repair**: O(k log n) where k = affected nodes

## Repair Strategies

1. **Lazy Repair** - Repair during queries
2. **Eager Repair** - Repair immediately
3. **Batch Repair** - Accumulate and repair periodically
4. **Background Repair** - Repair in background thread

## Related Modules

- [ruvector-delta-core](../ruvector-delta-core/CLAUDE.md) - Core delta types
- [ruvector-delta-graph](../ruvector-delta-graph/CLAUDE.md) - Graph deltas
- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Graph neural networks

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented delta-aware HNSW architecture
- Identified repair strategies and features
- Catalogued performance characteristics
