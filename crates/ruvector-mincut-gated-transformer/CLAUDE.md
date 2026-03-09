[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-mincut-gated-transformer**

# ruvector-mincut-gated-transformer

> **Ultra Low Latency Transformer with Mincut-Gated Coherence**
| 使用 min-cut 门控的超低延迟 Transformer 推理

## 模块职责

`ruvector-mincut-gated-transformer` 实现了基于 min-cut 分区的门控 Transformer，通过将计算图智能分区到不同计算单元（CPU、GPU、ANE），实现超低延迟推理。该模块特别适合边缘部署和实时应用。

### 核心功能

- **Min-cut 分区**: 智能计算图分区
- **门控机制**: 能量门控策略
- **稀疏注意力**: Mincut 感知的稀疏注意力
- **超低延迟**: 优化延迟至 <100μs
- **多种注意力**: Spike、Spectral PE、Linear Attention

## 入口与启动

### 库入口

```rust
// src/lib.rs (如果存在)
pub use transformer::MincutGatedTransformer;
pub use gate::{EnergyGate, GatePolicy};
pub use attention::{SpikeAttention, SpectralPositionEncoding};
```

### 初始化示例

```rust
use ruvector_mincut_gated_transformer::{MincutGatedTransformer, TransformerConfig};

let config = TransformerConfig {
  num_layers: 6,
  hidden_dim: 512,
  num_heads: 8,
  enable_spike_attention: true,
  enable_spectral_pe: true,
  gate_policy: GatePolicy::EnergyBased,
};

let transformer = MincutGatedTransformer::new(config)?;
let output = transformer.forward(input)?;
```

## 对外接口

### MincutGatedTransformer

```rust
pub struct MincutGatedTransformer {
    config: TransformerConfig,
    layers: Vec<MincutGatedLayer>,
    gate_policy: GatePolicy,
}

impl MincutGatedTransformer {
    pub fn new(config: TransformerConfig) -> Result<Self>;
    pub fn forward(&self, input: &Tensor) -> Result<Tensor>;
    pub fn forward_with_gate_info(&self, input: &Tensor) -> Result<(Tensor, GateInfo)>;
    pub fn get_partition_stats(&self) -> PartitionStats;
}
```

### EnergyGate

```rust
pub struct EnergyGate {
    threshold: f32,
    policy: GatePolicy,
}

impl EnergyGate {
    pub fn new(threshold: f32, policy: GatePolicy) -> Self;
    pub fn should_compute(&self, energy: f32) -> bool;
    pub fn update_threshold(&mut self, new_threshold: f32);
}

pub enum GatePolicy {
    EnergyBased,
    CoherenceBased,
    Hybrid,
}
```

### SpikeAttention

```rust
pub struct SpikeAttention {
    num_heads: usize,
    head_dim: usize,
    spike_threshold: f32,
}

impl SpikeAttention {
    pub fn new(num_heads: usize, head_dim: usize) -> Self;
    pub fn forward(&self, query: &Tensor, key: &Tensor, value: &Tensor) -> Result<Tensor>;
    pub fn energy_reduction_ratio(&self) -> f32;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `thiserror` | 2.0 | 错误处理 |
| `serde` | 1.0 | 序列化 |
| `getrandom` | 0.2 | 随机数（WASM） |

### 特性标志

- `default`: 启用 sliding_window
- `full`: 启用所有特性（simd、trace、linear_attention、int4、fixed_point_softmax、rmsnorm、spike_attention、spectral_pe、sparse_attention、energy_gate）
- `sliding_window`: 滑动窗口注意力
- `linear_attention`: 线性注意力
- `spike_attention`: Spike 驱动注意力
- `spectral_pe`: 谱位置编码
- `sparse_attention`: 稀疏注意力
- `energy_gate`: 能量门控
- `simd`: SIMD 加速
- `int4`: INT4 量化
- `fixed_point_softmax`: 定点 softmax
- `rmsnorm`: RMS 归一化
- `trace`: 跟踪/调试
- `wasm`: WASM 支持
- `no_std_gateway`: 无 std 网关支持

### 配置参数

```rust
pub struct TransformerConfig {
    pub num_layers: usize,
    pub hidden_dim: usize,
    pub num_heads: usize,
    pub intermediate_dim: usize,

    // 注意力配置
    pub attention_window: usize,
    pub sliding_window: Option<usize>,

    // 门控配置
    pub gate_threshold: f32,
    pub gate_policy: GatePolicy,

    // 量化配置
    pub quantization_bits: Option<usize>,
    pub enable_fixed_point_softmax: bool,
}
```

## 数据模型

### Tensor

```rust
pub struct Tensor {
    pub data: Vec<f32>,
    pub shape: Vec<usize>,
}
```

### GateInfo

```rust
pub struct GateInfo {
    pub total_gates: usize,
    pub active_gates: usize,
    pub energy_saved: f32,
    pub latency_reduction: f32,
}
```

### PartitionStats

```rust
pub struct PartitionStats {
    pub num_partitions: usize,
    pub partition_sizes: Vec<usize>,
    pub cross_partition_edges: usize,
    pub load_balance: f32,
}
```

## 测试与质量

### 基准测试

```
benches/
├── latency.rs          # 延迟基准
├── gate.rs             # 门控基准
└── kernel.rs           # 内核基准
```

### 示例

```
examples/
└── scorer.rs           # 评分器示例
```

### 测试运行

```bash
# 运行测试
cargo test -p ruvector-mincut-gated-transformer

# 运行基准测试
cargo bench -p ruvector-mincut-gated-transformer

# 运行特定特性
cargo test -p ruvector-mincut-gated-transformer --features spike_attention
```

### 质量指标

- 延迟基准：目标 <100μs
- 能耗基准：与稠密模型对比
- 门控效率：跳过计算的比例
- 准确性：与原始 Transformer 的差异

## 常见问题 (FAQ)

### Q1: Spike Attention 如何节能？

Spike Attention 只在输入变化超过阈值时才触发计算。对于静态或缓慢变化的输入，可以跳过 87% 的计算（Yao et al., 2023）。

### Q2: Spectral PE 的优势？

谱位置编码使用拉普拉斯矩阵的特征向量，比标准 sinusoidal PE 更好地捕捉图的拓扑结构。对于结构化数据特别有效。

### Q3: 何时使用 Min-cut 分区？

- **边缘设备**: 内存受限，需要智能卸载
- **混合计算**: CPU+GPU+ANE 异构计算
- **实时应用**: 需要保证延迟 SLA

## 相关文件清单

### 构建文件

```
Cargo.toml             # 依赖和特性配置
README.md              # 项目文档
```

### 基准测试

```
benches/
├── latency.rs
├── gate.rs
└── kernel.rs
```

### 示例

```
examples/
└── scorer.rs
```

## 相关模块

- **ruvector-mincut**: Min-cut 核心算法
- **ruvector-mincut-gated-transformer-wasm**: WASM 绑定
- **ruvector-attention**: 注意力机制
- **ruvector-solver**: 子线性求解

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录门控 Transformer 架构
- 完成特性标志和配置文档
