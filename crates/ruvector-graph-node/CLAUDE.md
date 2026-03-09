# ruvector-graph-node

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-graph-node**

## Module Description

Node.js bindings for RuVector Graph Database via NAPI-RS, providing Neo4j-compatible graph operations with Cypher query language support to JavaScript/TypeScript applications. This module enables native-speed graph database operations in Node.js.

## Entry Points

- **Main Library**: `src/lib.rs` - NAPI-RS bindings for graph operations
- **Build**: `build.rs` - NAPI build configuration

## Architecture

### Core Bindings

1. **Graph Operations**
   - Node creation and management
   - Edge creation and management
   - Graph traversal
   - Pattern matching

2. **Cypher Query Language**
   - Query parsing and execution
   - Result streaming
   - Transaction support
   - Prepared statements

3. **Storage Integration**
   - Persistent storage
   - Memory-mapped files
   - Compression support
   - Transaction management

4. **Hybrid Features**
   - Vector similarity search
   - GNN integration
   - RAG support
   - Semantic search

## Key Dependencies

- `ruvector-core` - Core vector operations
- `ruvector-graph` - Graph database engine (storage feature)
- `napi` - NAPI-RS framework
- `napi-derive` - NAPI derive macros
- `tokio` - Async runtime
- `futures` - Async utilities
- `thiserror` - Error handling
- `anyhow` - Error handling
- `tracing` - Logging
- `serde` - Serialization
- `serde_json` - JSON serialization
- `uuid` - UUID support

## Build Configuration

```toml
[lib]
crate-type = ["cdylib"]

[profile.release]
lto = true
strip = true
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
const { GraphDB } = require('ruvector-graph-node');

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

// Transaction support
const tx = db.beginTransaction();
try {
    const node1 = tx.createNode('Person', { name: 'Charlie' });
    const node2 = tx.createNode('Person', { name: 'Diana' });
    tx.createEdge(node1, node2, 'FRIENDS_WITH');
    tx.commit();
} catch (error) {
    tx.rollback();
}
```

## Cypher Query Support

```javascript
// Pattern matching
db.cypher('MATCH (n)-[r]->(m) RETURN n, r, m');

// Filtering
db.cypher('MATCH (n:Person) WHERE n.age > 25 RETURN n');

// Aggregation
db.cypher('MATCH (n:Person) RETURN count(n)');

// Path queries
db.cypher('MATCH path = (a)-[*1..3]-(b) RETURN path');

// Vector similarity (hybrid)
db.cypher(`
    MATCH (n:Document)
    WHERE vector.similarity(n.embedding, $query) > 0.8
    RETURN n
`, { query: embedding });
```

## Hybrid Features

```javascript
// Vector similarity search
const similar = db.searchByVector(
    embedding,
    'Document',
    10  // top-k
);

// RAG integration
const context = db.ragRetrieve(
    queryEmbedding,
    'knowledge_graph',
    5
);

// GNN inference
const predictions = db.gnnPredict(
    nodeIds,
    'link_prediction'
);
```

## Async Operations

```javascript
// Async operations with tokio
const nodes = await db.cypherAsync('MATCH (n) RETURN n');
const created = await db.createNodeAsync('Person', properties);
```

## Related Modules

- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database engine
- [ruvector-core](../ruvector-core/CLAUDE.md) - Core vector operations
- [ruvector-node](../ruvector-node/CLAUDE.md) - Core Node.js bindings
- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - Graph neural networks

## NPM Package

This module is published to npm as `@ruvector/graph`.

## Performance Characteristics

- **Query**: Neo4j-compatible Cypher execution
- **Storage**: Persistent with Redb backend
- **Vector**: Integrated similarity search
- **GNN**: Neural network inference
- **Parallel**: Multi-core query execution

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented NAPI-RS graph bindings
- Identified Cypher support and hybrid features
- Catalogued usage examples and performance
