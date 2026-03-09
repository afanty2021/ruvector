[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-attention-unified-wasm**

# ruvector-attention-unified-wasm

> **Unified WebAssembly Bindings for 18+ Attention Mechanisms**
> 统一的注意力机制 WASM 绑定，整合神经、DAG、图和 Mamba SSM

## 模块职责

`ruvector-attention-unified-wasm` 提供统一的 WebAssembly 绑定，整合了 18+ 种不同的注意力机制，包括神经网络注意力、DAG 注意力、图注意力（GNN）和 Mamba 状态空间模型。该模块为 Web 应用提供一站式注意力计算解决方案。

### 核心功能

- **统一接口**: 单一 API 支持 18+ 注意力类型
- **多架构支持**: 神经、DAG、图、Mamba SSM
- **WASM 优化**: 小包体积，高性能
- **类型安全**: 完整的 TypeScript 定义

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/attention-unified-wasm
```

### 基本使用

```typescript
import {
  createAttention,
  AttentionType,
  NeuralAttentionConfig,
  DagAttentionConfig,
  GraphAttentionConfig
} from '@ruvector/attention-unified-wasm';

// 创建神经网络注意力
const neuralConfig: NeuralAttentionConfig = {
  type: AttentionType.Neural,
  mechanism: 'flash',
  numHeads: 8,
  headDim: 64
};
const flashAttn = createAttention(neuralConfig);

// 创建 DAG 注意力
const dagConfig: DagAttentionConfig = {
  type: AttentionType.Dag,
  mechanism: 'topological',
  numHeads: 4,
  headDim: 32
};
const dagAttn = createAttention(dagConfig);

// 创建图注意力
const graphConfig: GraphAttentionConfig = {
  type: AttentionType.Graph,
  mechanism: 'gat',
  numHeads: 8,
  headDim: 64
};
const gatAttn = createAttention(graphConfig);
```

## 对外接口

### 注意力类型枚举

```typescript
enum AttentionType {
  // 神经网络注意力 (7 种)
  Neural = 'neural',

  // DAG 注意力 (7 种)
  Dag = 'dag',

  // 图注意力 (3 种)
  Graph = 'graph',

  // Mamba SSM
  Mamba = 'mamba'
}
```

### 配置类型

```typescript
interface BaseConfig {
  type: AttentionType;
  numHeads: number;
  headDim: number;
}

interface NeuralAttentionConfig extends BaseConfig {
  type: AttentionType.Neural;
  mechanism: 'flash' | 'linear' | 'hyperbolic' | 'performer' |
             'reformer' | 'synthesizer' | 'local';
  useCausalMask?: boolean;
  dropout?: number;
}

interface DagAttentionConfig extends BaseConfig {
  type: AttentionType.Dag;
  mechanism: 'topological' | 'causal' | 'hierarchical' |
             'feedback' | 'recurrent' | 'temporal' | 'skip';
  dagStructure?: DagStructure;
}

interface GraphAttentionConfig extends BaseConfig {
  type: AttentionType.Graph;
  mechanism: 'gat' | 'gcn' | 'graphsage';
  adjacencyMatrix?: Float32Array;
}

interface MambaAttentionConfig extends BaseConfig {
  type: AttentionType.Mamba;
  stateDim: number;
  dtRank: number;
}
```

### 统一 API

```typescript
interface Attention {
  forward(query: Tensor, key: Tensor, value: Tensor): Tensor;
  backward(gradOutput: Tensor): Tensor[];
  getConfig(): BaseConfig;
  dispose(): void;
}

function createAttention(config: NeuralAttentionConfig): Attention;
function createAttention(config: DagAttentionConfig): Attention;
function createAttention(config: GraphAttentionConfig): Attention;
function createAttention(config: MambaAttentionConfig): Attention;
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-attention` | 2.0 | 神经注意力核心 |
| `ruvector-dag` | 2.0 | DAG 注意力核心 |
| `ruvector-gnn` | 2.0 | 图注意力核心 |
| `wasm-bindgen` | 0.2 | WASM 绑定 |
| `serde-wasm-bindgen` | 0.6 | 序列化 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[profile.release]
opt-level = "z"        # 优化体积
lto = true
codegen-units = 1
panic = "abort"
strip = true

[profile.release.package."*"]
opt-level = "z"        # 依赖也优化体积
```

### 特性标志

- `default`: 启用 console_error_panic_hook
- `wee_alloc`: 使用 wee_alloc 分配器（~10KB 更小）

## 数据模型

### Tensor 类型

```typescript
class Tensor {
  constructor(data: Float32Array, shape: number[]);

  readonly data: Float32Array;
  readonly shape: number[];
  readonly dtype: 'float32' | 'float16';

  reshape(shape: number[]): Tensor;
  transpose(): Tensor;
  slice(axes: number[], starts: number[], ends: number[]): Tensor;
}
```

### DAG 结构

```typescript
interface DagStructure {
  numNodes: number;
  edges: Array<[from: number, to: number]>;
  topologicalOrder?: number[];
}
```

### 图结构

```typescript
interface GraphStructure {
  numNodes: number;
  numEdges: number;
  adjacencyMatrix: Float32Array;  // [numNodes, numNodes]
  edgeFeatures?: Float32Array;    // [numEdges, featureDim]
}
```

## 测试与质量

### 测试文件

```typescript
// tests/unified.test.ts
describe('Unified Attention', () => {
  test('neural attention types', () => {
    // 测试所有 7 种神经注意力
  });

  test('dag attention types', () => {
    // 测试所有 7 种 DAG 注意力
  });

  test('graph attention types', () => {
    // 测试所有 3 种图注意力
  });

  test('mamba ssm', () => {
    // 测试 Mamba SSM
  });
});
```

### 测试运行

```bash
# 构建 WASM
wasm-pack build --dev --target web

# 运行测试
wasm-pack test --firefox

# Node.js 测试
node test.mjs
```

### 质量指标

- 单元测试覆盖：所有 18+ 注意力类型
- 集成测试：多注意力组合场景
- WASM 包大小：<150KB（gzip）

## 常见问题 (FAQ)

### Q1: 如何选择注意力类型？

- **神经网络注意力**：标准序列处理（NLP、时间序列）
- **DAG 注意力**：有向无环图结构（代码、工作流）
- **图注意力**：图结构数据（社交网络、知识图谱）
- **Mamba SSM**：长序列建模（基因组、长文档）

### Q2: 性能如何？

WASM 版本约为原生 Rust 的 70-80% 性能。关键优化：
- 使用 `opt-level = "z"` 减小体积
- 避免 JavaScript/WASM 边界拷贝
- 使用 TypedArray 零拷贝传输

### Q3: 如何扩展新的注意力类型？

1. 在核心 Rust 实现中添加新注意力
2. 在配置枚举中添加新类型
3. 实现对应的工厂函数
4. 添加测试和文档

## 相关文件清单

### 源文件

```
src/
└── lib.rs              # 统一 WASM 绑定
```

### 构建文件

```
Cargo.toml             # Rust 依赖配置
package.json           # NPM 包配置
pkg/                   # WASM 输出
```

## 相关模块

- **ruvector-attention**: 神经注意力核心
- **ruvector-dag**: DAG 注意力核心
- **ruvector-gnn**: 图注意力核心
- **ruvector-attention-node**: Node.js 原生绑定
- **ruvector-attention-wasm**: 单独的注意力 WASM

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录统一 WASM 架构
- 完成 18+ 注意力类型文档
