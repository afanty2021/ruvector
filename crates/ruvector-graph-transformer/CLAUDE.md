[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-graph-transformer**

# ruvector-graph-transformer

> **Unified Graph Transformer with Proof-Gated Mutation**
> 8 个验证模块支持物理、生物、流形、时间和经济图智能

## 模块职责

`ruvector-graph-transformer` 实现统一的图 Transformer 架构，整合了 8 个经过验证的模块，涵盖物理仿真、生物建模、自组织、流形学习、时间序列和经济建模等领域。该模块使用证明门控的突变基底，确保状态转换的可验证性。

### 核心功能

- **统一架构**: 单一框架支持多种图 Transformer 类型
- **8 个验证模块**: 物理验证、生物验证、自组织、验证训练、流形、时间、经济、子线性注意力
- **证明门控**: 可验证的状态转换
- **子线性复杂度**: 高效的大图处理

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use transformer::{GraphTransformer, TransformerConfig};
pub use proof_gated::{ProofGatedLayer, ProofVerifier};
pub use verified_training::VerifiedTrainer;
```

### 初始化示例

```rust
use ruvector_graph_transformer::{GraphTransformer, TransformerConfig};

let config = TransformerConfig {
    num_heads: 8,
    hidden_dim: 512,
    num_layers: 6,
    enable_physics: true,
    enable_biological: true,
    enable_temporal: true,
};

let transformer = GraphTransformer::new(config)?;
let output = transformer.forward(graph, features)?;
```

## 对外接口

### GraphTransformer

```rust
pub struct GraphTransformer {
    config: TransformerConfig,
    layers: Vec<TransformerLayer>,
    proof_verifier: ProofVerifier,
}

impl GraphTransformer {
    pub fn new(config: TransformerConfig) -> Result<Self>;
    pub fn forward(&self, graph: &Graph, features: &Features) -> Result<Output>;
    pub fn train(&mut self, dataset: &Dataset) -> Result<TrainingMetrics>;
    pub fn verify_state(&self, proof: &Proof) -> bool;
}
```

### TransformerConfig

```rust
pub struct TransformerConfig {
    pub num_heads: usize,
    pub hidden_dim: usize,
    pub num_layers: usize,
    pub dropout: f32,

    // 模块启用标志
    pub enable_physics: bool,
    pub enable_biological: bool,
    pub enable_self_organizing: bool,
    pub enable_verified_training: bool,
    pub enable_manifold: bool,
    pub enable_temporal: bool,
    pub enable_economic: bool,
    pub enable_sublinear: bool,
}
```

### ProofGatedLayer

```rust
pub struct ProofGatedLayer {
    inner_layer: Box<dyn Layer>,
    proof_generator: ProofGenerator,
}

impl ProofGatedLayer {
    pub fn forward_with_proof(&self, input: &Tensor) -> (Tensor, Proof);
    pub fn verify_transition(&self, input: &Tensor, output: &Tensor, proof: &Proof) -> bool;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-verified` | 0.1 | 验证框架 |
| `ruvector-gnn` | 2.0 | 图神经网络 |
| `ruvector-attention` | 2.0 | 注意力机制 |
| `ruvector-mincut` | 2.0 | Min-cut 算法 |
| `ruvector-solver` | 2.0 | 子线性求解 |
| `ruvector-coherence` | 2.0 | 一致性检查 |
| `serde` | workspace | 序列化 |
| `rand` | workspace | 随机数 |

### 特性标志

- `default`: 启用 sublinear 和 verified-training
- `sublinear`: 子线性注意力
- `physics`: 物理模块
- `biological`: 生物模块
- `self-organizing`: 自组织模块
- `verified-training`: 验证训练
- `manifold`: 流形模块
- `temporal`: 时间模块
- `economic`: 经济模块
- `full`: 启用所有模块

### 配置参数

```rust
pub struct ModuleConfig {
    // 物理模块
    pub physics_time_step: f32,
    pub physics_integrator: IntegratorType,

    // 生物模块
    pub biological_mutation_rate: f32,
    pub biological_selection_pressure: f32,

    // 时间模块
    pub temporal_window_size: usize,
    pub temporal_decay: f32,

    // 经济模块
    pub economic_budget: f64,
    pub economic_cost_function: CostFunction,
}
```

## 数据模型

### Graph

```rust
pub struct Graph {
    pub num_nodes: usize,
    pub num_edges: usize,
    pub adjacency: SparseMatrix,
    pub edge_features: Option<Tensor>,
    pub node_features: Option<Tensor>,
}
```

### Proof

```rust
pub struct Proof {
    pub state_hash: [u8; 32],
    pub transition_hash: [u8; 32],
    pub signature: Signature,
    pub metadata: ProofMetadata,
}
```

### TrainingMetrics

```rust
pub struct TrainingMetrics {
    pub loss: f32,
    pub accuracy: f32,
    pub verification_rate: f32,
    pub convergence_metrics: ConvergenceMetrics,
}
```

## 测试与质量

### 测试文件

```
tests/
└── integration.rs     # 集成测试
```

### 测试运行

```bash
# 运行所有测试
cargo test -p ruvector-graph-transformer

# 运行特定模块测试
cargo test -p ruvector-graph-transformer --features physics

# 运行全功能测试
cargo test -p ruvector-graph-transformer --features full
```

### 质量指标

- 单元测试覆盖：各个模块的独立测试
- 集成测试：多模块协同测试
- 属性测试：使用 proptest 验证不变性
- 基准测试：大图性能测试

## 常见问题 (FAQ)

### Q1: 如何选择启用哪些模块？

根据应用场景：
- **物理仿真**: 启用 `physics`、`temporal`
- **生物建模**: 启用 `biological`、`self-organizing`
- **金融预测**: 启用 `economic`、`temporal`
- **推荐系统**: 启用 `manifold`、`sublinear`
- **通用场景**: 使用 `full` 启用所有模块

### Q2: 证明门控如何工作？

每一层转换都生成一个证明，包含：
1. 输入状态的哈希
2. 输出状态的哈希
3. 转换参数的签名
4. 元数据（时间戳、模块信息）

验证者可以独立验证状态转换的正确性。

### Q3: 性能如何？

子线性注意力机制使复杂度接近 O(n log n) 而非 O(n²)。大图（百万节点）仍能高效处理。

## 相关文件清单

### 源文件

```
src/
├── lib.rs                  # 库入口
├── config.rs               # 配置定义
├── error.rs                # 错误类型
├── transformer.rs          # 主 Transformer
├── proof_gated.rs          # 证明门控层
├── physics.rs              # 物理模块
├── biological.rs           # 生物模块
├── self_organizing.rs      # 自组织模块
├── verified_training.rs    # 验证训练
├── manifold.rs             # 流形模块
├── temporal.rs             # 时间模块
├── economic.rs             # 经济模块
└── sublinear_attention.rs  # 子线性注意力
```

### 测试文件

```
tests/
└── integration.rs
```

## 相关模块

- **ruvector-gnn**: 图神经网络核心
- **ruvector-attention**: 注意力机制
- **ruvector-mincut**: Min-cut 分区
- **ruvector-verified**: 验证框架
- **ruvector-solver**: 子线性求解

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 8 个验证模块架构
- 完成统一 Transformer API 文档
