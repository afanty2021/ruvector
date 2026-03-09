# ruvector-gnn-wasm

[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-gnn-wasm**

## Module Description

WebAssembly bindings for RuVector GNN with tensor compression and differentiable search, providing browser-compatible graph neural network operations for web applications. This module enables high-performance GNN inference in web browsers without server dependencies.

## Entry Points

- **Main Library**: `src/lib.rs` - WASM bindings for GNN operations

## Architecture

### Core Components

1. **GNN Operations**
   - Graph construction
   - Message passing
   - Aggregation functions
   - Readout operations

2. **Tensor Operations**
   - Compressed tensor storage
   - Efficient computation
   - Memory management
   - SIMD acceleration

3. **Differentiable Search**
   - Differentiable graph search
   - Gradient flow through search
   - Neural guided search
   - Learning to search

4. **Optimization**
   - Size-optimized bundle
   - Fast inference
   - Low memory footprint
   - Browser compatibility

## Key Dependencies

- `ruvector-gnn` - GNN engine (wasm feature, default-features = false)
- `wasm-bindgen` - WASM bindings
- `js-sys` - JavaScript system bindings
- `getrandom` - Random number generation
- `serde` - Serialization
- `serde-wasm-bindgen` - Serde WASM bridge
- `console_error_panic_hook` - Panic handling (optional)

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
- **console_error_panic_hook** - Better error messages in console

## Testing

- **Unit Tests**: Located throughout the codebase in `tests` modules
- **WASM Tests**: Use `wasm-bindgen-test` for browser testing

## Usage Examples

```javascript
import { GNNModel, GraphLayer } from 'ruvector-gnn-wasm';

// Create GNN model
const model = new GNNModel({
    inputDim: 128,
    hiddenDim: 64,
    outputDim: 32,
    numLayers: 2
});

// Create graph (COO format)
const graph = {
    numNodes: 100,
    edgeIndex: [
        new Int32Array([0, 1, 2, ...]),  // Source nodes
        new Int32Array([1, 2, 3, ...])   // Target nodes
    ],
    nodeFeatures: new Float32Array(/* features */)
};

// Forward pass
const output = model.forward(graph);

// Node classification
const predictions = model.predictNodeClasses(graph);

// Node embedding
const embeddings = model.generateNodeEmbeddings(graph);

// Link prediction
const linkProb = model.predictLink(graph, 0, 1);
```

## Tensor Compression

```javascript
// Compress tensors for storage
const compressed = model.compressTensors();

// Decompress for inference
model.decompressTensors(compressed);

// Save compressed model
const modelData = model.saveCompressed();
localStorage.setItem('gnn-model', modelData);

// Load compressed model
const loaded = GNNModel.loadCompressed(modelData);
```

## Differentiable Search

```javascript
// Differentiable shortest path
const path = model.differentiableShortestPath(graph, source, target);

// Differentiable page rank
const ranks = model.differentiablePageRank(graph);

// Neural guided search
const result = model.neuralGuidedSearch(graph, query, guidance);
```

## Web Worker Support

```javascript
// Main thread
const worker = new Worker('gnn-worker.js');
worker.postMessage({ type: 'inference', graph });
worker.onmessage = (e) => {
    const output = e.data;
    console.log('GNN output:', output);
};

// Worker thread (gnn-worker.js)
import { GNNModel } from 'ruvector-gnn-wasm';
const model = new GNNModel(config);
self.onmessage = async (e) => {
    if (e.data.type === 'inference') {
        const output = model.forward(e.data.graph);
        self.postMessage(output);
    }
};
```

## Browser Compatibility

- **Chrome**: 57+
- **Firefox**: 52+
- **Safari**: 11+
- **Edge**: 16+

## Bundle Size

- **Base**: ~85KB (gzipped)
- **With Compression**: ~90KB (gzipped)

## Related Modules

- [ruvector-gnn](../ruvector-gnn/CLAUDE.md) - GNN engine
- [ruvector-wasm](../ruvector-wasm/CLAUDE.md) - Core WASM bindings
- [ruvector-graph-wasm](../ruvector-graph-wasm/CLAUDE.md) - Graph WASM

## Use Cases

1. **Interactive Visualization** - Real-time GNN inference
2. **Educational Tools** - Graph neural network learning
3. **Recommendation Systems** - Client-side recommendations
4. **Knowledge Graphs** - Browser-based graph learning
5. **Edge Computing** - GNN inference at the edge

## Performance Characteristics

- **Inference**: Fast GNN inference in browser
- **Memory**: Compressed tensor storage
- **Size**: Optimized for small bundle size
- **Parallel**: Web Worker support

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented WASM GNN bindings
- Identified tensor compression and differentiable search
- Catalogued browser compatibility and use cases
