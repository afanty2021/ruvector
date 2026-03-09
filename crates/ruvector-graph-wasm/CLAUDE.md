# ruvector-graph-wasm

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-graph-wasm**

## Module Description

WebAssembly bindings for RuVector graph database with Neo4j-inspired API and Cypher support, providing browser-compatible graph operations for web applications without server dependencies.

## Entry Points

- **Main Library**: `src/lib.rs` - WASM bindings for graph operations

## Architecture

### Core Components

1. **Graph Operations**
   - Node creation and management
   - Edge creation and management
   - Graph traversal
   - Pattern matching

2. **Cypher Query Language**
   - Query parsing and execution
   - Basic Cypher support
   - Result streaming
   - Prepared queries

3. **IndexedDB Integration**
   - Persistent storage
   - Browser-native database
   - Transaction support
   - Async operations

4. **Web Worker Support**
   - Parallel graph processing
   - Background computation
   - Message passing

## Key Dependencies

- `ruvector-core` - Core vector operations (default-features = false)
- `ruvector-graph` - Graph database (wasm feature, default-features = false)
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
- `uuid` - UUID support
- `regex` - Regular expressions (for basic Cypher parsing)

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
    "Worker",
    "MessagePort",
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

[package.metadata.wasm-pack.profile.release]
wasm-opt = false     # Skip wasm-opt for faster builds
```

## Features

- **default** - Basic WASM functionality
- **simd** - Enable SIMD optimizations

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **WASM Tests**: Use `wasm-bindgen-test` for browser testing

## Usage Examples

```javascript
import { GraphDB } from 'ruvector-graph-wasm';

// Create graph database
const db = new GraphDB();

// Create nodes
const alice = db.createNode('Person', { name: 'Alice', age: 30 });
const bob = db.createNode('Person', { name: 'Bob', age: 25 });

// Create relationship
db.createEdge(alice, bob, 'KNOWS', { since: 2020 });

// Execute Cypher query
const result = db.cypher(`
    MATCH (a:Person)-[r:KNOWS]->(b:Person)
    WHERE a.name = 'Alice'
    RETURN a, r, b
`);

// Use IndexedDB for persistence
await db.persist('my-graph-db');

// Query with patterns
const friends = db.cypher(`
    MATCH (a:Person)-[r:FRIENDS_WITH]->(b:Person)
    RETURN a.name, b.name
`);
```

## Cypher Support

### Supported Operations

1. **MATCH** - Pattern matching
   ```cypher
   MATCH (n)-[r]->(m) RETURN n, r, m
   ```

2. **WHERE** - Filtering
   ```cypher
   MATCH (n:Person) WHERE n.age > 25 RETURN n
   ```

3. **CREATE** - Create nodes and edges
   ```cypher
   CREATE (n:Person { name: 'Charlie' })
   ```

4. **RETURN** - Return results
   ```cypher
   MATCH (n:Person) RETURN n.name
   ```

### Limitations (WASM)

- No full Cypher parser (regex-based)
- Limited to basic queries
- No transactions (uses IndexedDB transactions)
- No distributed features

## Web Worker Usage

```javascript
// Main thread
const worker = new Worker('graph-worker.js');
worker.postMessage({ type: 'query', cypher: 'MATCH (n) RETURN n' });
worker.onmessage = (e) => {
    const results = e.data;
    console.log('Query results:', results);
};

// Worker thread (graph-worker.js)
import { GraphDB } from 'ruvector-graph-wasm';
const db = new GraphDB();
self.onmessage = async (e) => {
    if (e.data.type === 'query') {
        const results = db.cypher(e.data.cypher);
        self.postMessage(results);
    }
};
```

## Browser Compatibility

- **Chrome**: 57+
- **Firefox**: 52+
- **Safari**: 11+
- **Edge**: 16+

## Bundle Size

- **Base**: ~75KB (gzipped)
- **With SIMD**: ~80KB (gzipped)

## Related Modules

- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database engine
- [ruvector-wasm](../ruvector-wasm/CLAUDE.md) - Core WASM bindings
- [ruvector-gnn-wasm](../ruvector-gnn-wasm/CLAUDE.md) - GNN WASM

## Use Cases

1. **Knowledge Graphs** - Browser-based knowledge graphs
2. **Social Networks** - Client-side relationship visualization
3. **Data Visualization** - Interactive graph displays
4. **Offline Applications** - PWA with local graph DB
5. **Educational Tools** - Graph theory learning

## Performance Characteristics

- **Query**: Basic Cypher execution (regex-based)
- **Storage**: IndexedDB persistence
- **Traversal**: In-memory graph operations
- **Workers**: Parallel processing support

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented WASM graph bindings
- Identified Cypher support and limitations
- Catalogued browser compatibility and use cases
