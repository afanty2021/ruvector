[根目录](../../CLAUDE.md) > **ruvector-postgres**

# ruvector-postgres - Module Documentation

## Module Responsibilities

PostgreSQL extension providing 230+ SQL functions for vector search, GNN layers, attention mechanisms, and self-learning capabilities. Drop-in replacement for pgvector with advanced AI features.

## Entry Points

- **`src/lib.rs`** - Main extension entry point
- **`src/index/`** - HNSW and IVFFlat index implementations
- **`src/gnn/`** - GNN layer operators
- **`src/attention/`** - Attention mechanism operators
- **`src/graph/cypher/`** - Cypher query language
- **`src/embeddings/`** - Local embedding generation

## Key Interfaces

### SQL Functions
```sql
-- Core vector operations
ruvector_cosine_distance(vector, vector) -> float8
ruvector_l2_distance(vector, vector) -> float8
ruvector_inner_product(vector, vector) -> float8

-- GNN layers
ruvector_gnn_forward(table_name, layer_config) -> setof record

-- Attention mechanisms
ruvector_flash_attention(query, keys, values) -> vector
ruvector_multi_head_attention(...) -> vector

-- Graph queries
cypher_query(query_text) -> setof record

-- Embeddings
ruvector_embed(text, model_name) -> vector
```

## Dependencies

### Required
- `pgrx` 0.12 - PostgreSQL extension framework
- `simsimd` 5.9 - SIMD distance calculations
- `rayon` 1.10 - Parallel processing

### Optional Features
- `fastembed` - Local embedding generation
- `ruvector-solver` - Sublinear solvers
- `ruvector-math` - Advanced math distances
- `ruvector-attention` - Extended attention (46 mechanisms)
- `ruvector-sona` - Self-optimizing neural architecture
- `ruvector-domain-expansion` - Transfer learning

## Data Models

### Vector Type
```sql
CREATE TYPE ruvector AS (
    dimensions INTEGER,
    data FLOAT8[]
);
```

### Index Types
- `ruhnsw` - HNSW index
- `ruivfflat` - IVFFlat index

## Testing

### Running Tests
```bash
# Unit tests
cargo test -p ruvector-postgres

# Integration tests (requires PostgreSQL)
cargo pgrx-test

# Specific feature
cargo test -p ruvector-postgres --features embeddings
```

## Key Files

| Directory | Description |
|-----------|-------------|
| `src/index/` | HNSW, IVFFlat implementations |
| `src/gnn/` | GCN, GraphSAGE, GAT operators |
| `src/attention/` | Flash, multi-head attention |
| `src/graph/cypher/` | Cypher parser and executor |
| `src/graph/sparql/` | SPARQL 1.1 support |
| `src/embeddings/` | Local embedding generation |
| `src/hyperbolic/` | Poincare, Lorentz distances |
| `src/hybrid/` - Hybrid search (vector + BM25) |
| `src/healing/` | Self-healing indexes |
| `src/dag/` | Neural DAG learning (59 functions) |
| `src/solver/` - Sublinear solvers |
| `src/math/` | Advanced distances |

## SQL Function Categories

### Core Operations (15+ functions)
- Distance metrics (cosine, L2, inner product, Manhattan)
- Vector operations (normalize, add, scalar mul)
- Aggregations (avg, sum, min, max)

### Hyperbolic Geometry (8 functions)
- Poincare distance
- Lorentz distance
- Hyperbolic normalization

### GNN Layers (12 functions)
- GCN forward pass
- GraphSAGE aggregation
- GAT attention weights

### Attention Mechanisms (20+ functions)
- Flash attention
- Multi-head attention
- Linear attention
- MoE routing

### Graph Queries (30+ functions)
- Cypher query execution
- SPARQL 1.1 support
- Graph traversal

### Learning (25+ functions)
- SONA micro-LoRA
- Trajectory learning
- EWC++ consolidation

### Sublinear Solvers (15+ functions)
- PageRank
- Conjugate gradient
- Laplacian solver
- Spectral clustering

## Feature Flags

### Postgres Versions
- `pg14`, `pg15`, `pg16`, `pg17` (default)

### SIMD
- `simd-auto` (default) - Runtime detection
- `simd-avx2`, `simd-avx512`, `simd-neon`

### Index Features
- `index-hnsw` - HNSW indexing
- `index-ivfflat` - IVFFlat indexing
- `index-all` - All indexes

### Quantization
- `quantization-scalar` - 4x compression
- `quantization-product` - 8-32x compression
- `quantization-binary` - 32x compression
- `quant-all` - All quantization

### AI Features
- `learning` - Self-learning / ReasoningBank
- `attention` - 39 attention mechanisms
- `gnn` - GNN layers
- `hyperbolic` - Hyperbolic embeddings
- `graph` - Graph operations
- `embeddings` - Local embedding generation

### v0.3 Features
- `solver` - Sublinear solvers
- `math-distances` - Wasserstein, Sinkhorn, KL
- `tda` - Topological data analysis
- `attention-extended` - 46 attention mechanisms
- `sona-learning` - SONA integration
- `domain-expansion` - Transfer learning

### Bundles
- `ai-complete` - All AI features
- `all-features-v3` - Everything

## Common Patterns

### Creating Extension
```sql
CREATE EXTENSION ruvector;
```

### Creating Table with Vectors
```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding ruvector(1536)
);
```

### Creating Index
```sql
CREATE INDEX ON documents USING ruhnsw (embedding ruvector_l2_ops);
```

### Searching
```sql
SELECT content, embedding <-> '[0.15, 0.25, ...]'::ruvector AS distance
FROM documents
ORDER BY distance
LIMIT 10;
```

### Local Embeddings
```sql
SELECT ruvector_embed('Hello world', 'BAAI/bge-small-en-v1.5');
```

### GNN Forward Pass
```sql
SELECT * FROM ruvector_gnn_forward(
    'my_table',
    '{"input_dim": 128, "output_dim": 64, "layer_type": "gcn"}'::jsonb
);
```

## Performance

### Distance Calculations
- SIMD: 80K+ QPS on 8 cores
- AVX-512: 2-3x faster than scalar

### Index Performance
- HNSW: <1ms at 1M vectors
- IVFFlat: ~5ms at 1M vectors

### Embedding Generation
- Local models: 10-50ms per document
- No API costs

## Related Modules

- **[ruvector-core](../ruvector-core/)** - Core engine
- **[ruvector-gnn](../ruvector-gnn/)** - GNN layers
- **[ruvector-attention](../ruvector-attention/)** - Attention mechanisms
- **[sona](../sona/)** - Self-optimizing

## Deployment

### Docker
```bash
docker run -d --name ruvector-pg \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  ruvnet/ruvector-postgres:latest
```

### From Source
```bash
cargo pgrx install --release
```

## Changelog

### 2026-03-09
- Initial module documentation generated
- Catalogued 230+ SQL functions
- Documented feature flags and bundles
