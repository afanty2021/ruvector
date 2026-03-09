# ruvector-gnn-node

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-gnn-node**

## Module Description

Node.js bindings for Ruvector GNN (Graph Neural Network) via NAPI-RS, providing high-performance graph learning operations to JavaScript/TypeScript applications. This module enables native-speed neural network inference and training in Node.js.

## Entry Points

- **Main Library**: `src/lib.rs` - NAPI-RS bindings for GNN operations
- **Build**: `build.rs` - NAPI build configuration

## Architecture

### Core Bindings

1. **Graph Operations**
   - Graph construction
   - Message passing
   - Aggregation functions
   - Readout operations

2. **Neural Layers**
   - GCN (Graph Convolutional Networks)
   - GAT (Graph Attention Networks)
   - GraphSAGE
   - Custom GNN architectures

3. **Training**
   - Forward pass
   - Backward pass
   - Gradient computation
   - Optimization

4. **Inference**
   - Node classification
   - Link prediction
   - Graph classification
   - Embedding generation

## Key Dependencies

- `ruvector-gnn` - GNN engine (default-features = false)
- `napi` - NAPI-RS framework
- `napi-derive` - NAPI derive macros
- `serde_json` - JSON serialization

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
const { GNNModel, GraphLayer } = require('ruvector-gnn-node');

// Create GNN model
const model = new GNNModel({
    inputDim: 128,
    hiddenDim: 64,
    outputDim: 32,
    numLayers: 3,
    layerType: 'GCN'  // or 'GAT', 'GraphSAGE'
});

// Create graph
const graph = {
    numNodes: 100,
    edgeIndex: [[0, 1, 2, ...], [1, 2, 3, ...]],  // COO format
    nodeFeatures: Float32Array.of(/* features */)
};

// Forward pass
const output = model.forward(graph);

// Node classification
const predictions = model.predictNodeClasses(graph);

// Link prediction
const linkProbabilities = model.predictLinks(graph, [[0, 1], [2, 3]]);

// Graph embedding
const embedding = model.generateGraphEmbedding(graph);

// Training (if supported)
const loss = model.train(graph, labels);
```

## Layer Types

1. **GCN (Graph Convolutional Network)**
   ```javascript
   const layer = new GraphLayer('GCN', {
       inputDim: 128,
       outputDim: 64
   });
   ```

2. **GAT (Graph Attention Network)**
   ```javascript
   const layer = new GraphLayer('GAT', {
       inputDim: 128,
       outputDim: 64,
       numHeads: 8
   });
   ```

3. **GraphSAGE**
   ```javascript
   const layer = new GraphLayer('GraphSAGE', {
       inputDim: 128,
       outputDim: 64,
       aggregator: 'mean'  // or 'max', 'sum'
   });
   ```

## Aggregation Functions

```javascript
// Mean aggregation
const meanAgg = model.setAggregation('mean');

// Max aggregation
const maxAgg = model.setAggregation('max');

// Sum aggregation
const sumAgg = model.setAggregation('sum');

// Attention aggregation
const attnAgg = model.setAggregation('attention');
```

## Training Workflow

```javascript
// Create training data
const trainGraph = loadGraph();
const labels = loadLabels();

// Set up optimizer
const optimizer = {
    type: 'Adam',
    learningRate: 0.001,
    weightDecay: 0.0005
};

// Training loop
for (let epoch = 0; epoch < 100; epoch++) {
    const loss = model.train(trainGraph, labels, optimizer);
    console.log(`Epoch ${epoch}: loss = ${loss}`);
}

// Save model
const modelData = model.save();
fs.writeFileSync('model.bin', modelData);
```

## Related Modules

- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - GNN engine
- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database
- [ruvector-node](../ruvector-node/CLAUDE.md) - Core Node.js bindings
- [ruvector-attention](../ruvector-attention/CLAUDE.md) - Attention mechanisms

## NPM Package

This module is published to npm as `@ruvector/gnn`.

## Performance Characteristics

- **Inference**: Native-speed GNN inference
- **Training**: GPU-accelerated when available
- **Memory**: Efficient tensor operations
- **Parallel**: Multi-core message passing

## Use Cases

1. **Node Classification** - Categorize nodes in graphs
2. **Link Prediction** - Predict future connections
3. **Graph Classification** - Classify entire graphs
4. **Recommendation Systems** - Graph-based recommendations
5. **Knowledge Graphs** - Knowledge completion

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented NAPI-RS GNN bindings
- Identified layer types and aggregation functions
- Catalogued usage examples and performance
