[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-sparse-inference**

# ruvector-sparse-inference

> **PowerInfer-Style Sparse Inference Engine**
> 高效稀疏神经网络推理，支持边缘设备部署

## 模块职责

`ruvector-sparse-inference` 实现 PowerInfer 风格的稀疏推理引擎，通过激活稀疏化和量化显著减少神经网络推理的计算开销。该模块特别适合边缘设备和资源受限环境，可提供接近稠密模型的准确性同时大幅降低延迟和能耗。

### 核心功能

- **激活稀疏化**: 仅计算活跃神经元，跳过无关计算
- **离线-在线预测**: 稀疏模式预测器决定激活路由
- **多后端支持**: CPU、NPU、WASM 后端
- **量化支持**: INT4/INT8 量化减少内存占用
- **GGUF 模型加载**: 支持标准 GGUF 格式

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use engine::SparseEngine;
pub use config::{EngineConfig, SparseConfig};
pub use model::ModelLoader;
pub use predictor::ActivationPredictor;
```

### 初始化示例

```rust
use ruvector_sparse_inference::{SparseEngine, EngineConfig};

let config = EngineConfig {
    backend: BackendType::Cpu,
    sparse_threshold: 0.3,
    enable_quantization: true,
    ..Default::default()
};

let engine = SparseEngine::new(config)?;
engine.load_model("model.gguf").await?;

let output = engine.forward(input).await?;
```

## 对外接口

### SparseEngine

```rust
pub struct SparseEngine {
    config: EngineConfig,
    model: Option<Model>,
    backend: Box<dyn Backend>,
    predictor: ActivationPredictor,
}

impl SparseEngine {
    pub fn new(config: EngineConfig) -> Result<Self>;
    pub async fn load_model(&mut self, path: &str) -> Result<()>;
    pub async fn forward(&mut self, input: Tensor) -> Result<Tensor>;
    pub fn stats(&self) -> EngineStats;
    pub fn reset_stats(&mut self);
}
```

### EngineConfig

```rust
pub struct EngineConfig {
    pub backend: BackendType,
    pub sparse_threshold: f32,
    pub enable_quantization: bool,
    pub cache_size: usize,
    pub num_threads: usize,
    pub offline_predictor: bool,
}

pub enum BackendType {
    Cpu,
    Npu,
    Wasm,
}
```

### ActivationPredictor

```rust
pub struct ActivationPredictor {
    offline_router: OfflineRouter,
    online_adapter: OnlineAdapter,
}

impl ActivationPredictor {
    pub fn predict_sparse(&self, x: &Tensor) -> SparseMask;
    pub fn update_online(&mut self, input: &Tensor, actual: &SparseMask);
    pub fn adaptation_rate(&self) -> f32;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ndarray` | 0.16 | 张量计算 |
| `rand` | workspace | 随机数生成 |
| `serde` | workspace | 序列化 |
| `rayon` | workspace | 并行处理 |
| `memmap2` | workspace | 内存映射 |
| `half` | 2.4 | 半精度浮点 |
| `byteorder` | 1.5 | 字节序处理 |

### 特性标志

无特定特性标志。

### 配置参数

```rust
pub struct SparseConfig {
    pub sparsity: f32,              // 目标稀疏度 (0-1)
    pub update_interval: usize,     // 在线更新间隔
    pub drift_threshold: f32,       // 漂移检测阈值
    pub cache_budget: usize,        // 缓存预算
}
```

## 数据模型

### SparseMask

```rust
pub struct SparseMask {
    pub active_indices: Vec<usize>,
    pub sparsity: f32,
    pub confidence: f32,
}

impl SparseMask {
    pub fn from_dense(mask: &[bool]) -> Self;
    pub fn to_dense(&self, size: usize) -> Vec<bool>;
    pub fn apply(&self, tensor: &mut Tensor);
}
```

### LayerActivation

```rust
pub struct LayerActivation {
    pub layer_id: usize,
    pub sparse_mask: SparseMask,
    pub compute_saved: f32,
    pub accuracy_impact: f32,
}
```

### QuantizedTensor

```rust
pub struct QuantizedTensor {
    pub data: Vec<u8>,
    pub scale: f32,
    pub zero_point: i32,
    pub dtype: QuantizedDtype,
}

pub enum QuantizedDtype {
    Int4,
    Int8,
}
```

## 测试与质量

### 测试文件

```
tests/
├── unit/
│   ├── predictor_tests.rs      # 预测器单元测试
│   └── quantization_tests.rs   # 量化测试
├── integration/
│   ├── model_loading_tests.rs  # 模型加载测试
│   └── sparse_inference_tests.rs # 推理集成测试
└── property/
    └── sparse_invariant.rs     # 稀疏性不变量
```

### 测试运行

```bash
# 运行所有测试
cargo test -p ruvector-sparse-inference

# 运行基准测试
cargo bench -p ruvector-sparse-inference

# 测试特定后端
cargo test -p ruvector-sparse-inference --features backend-cpu
```

### 质量指标

- 单元测试覆盖：预测器、量化、稀疏操作
- 集成测试覆盖：端到端推理流程
- 属性测试：稀疏性保证、准确性验证
- 基准测试：延迟、吞吐量、能耗

## 常见问题 (FAQ)

### Q1: 如何平衡稀疏度和准确性？

通过 `sparse_threshold` 参数控制稀疏度。较激进的稀疏化（高阈值）可以节省更多计算但可能影响准确性。建议从 0.3-0.5 开始，根据具体模型调整。

### Q2: 在线预测器如何适应？

在线预测器持续监控实际激活模式，当检测到漂移（通过 `drift_threshold`）时更新路由策略。适应速度由 `update_interval` 控制。

### Q3: 支持哪些模型格式？

目前支持 GGUF 格式的模型加载。GGUF 提供高效的内存映射和量化支持，适合大模型部署。

## 相关文件清单

### 源文件

```
src/
├── lib.rs                  # 库入口
├── config.rs               # 配置结构
├── error.rs                # 错误类型
├── engine.rs               # 主引擎（如存在）
├── backend/
│   ├── mod.rs              # 后端接口
│   ├── cpu.rs              # CPU 后端
│   ├── npu.rs              # NPU 后端
│   └── wasm.rs             # WASM 后端
├── model/
│   ├── mod.rs              # 模型接口
│   ├── gguf.rs             # GGUF 加载器
│   ├── loader.rs           # 模型加载器
│   ├── runners.rs          # 模型运行器
│   └── types.rs            # 模型类型
├── predictor/
│   ├── mod.rs              # 预测器接口
│   └── lowrank.rs          # 低秩预测
├── sparse/
│   ├── mod.rs              # 稀疏接口
│   └── ffn.rs              # 稀疏 FFN
├── pi/
│   ├── mod.rs              # PI 模块根
│   ├── angular.rs          # 角度相关
│   ├── chaos.rs            # 混沌相关
│   ├── constants.rs        # 常量定义
│   └── drift.rs            # 漂移检测
├── precision/
│   ├── mod.rs              # 精度模块根
│   ├── lanes.rs            # 精度通道
│   ├── policy.rs           # 精度策略
│   ├── quantizers.rs       # 量化器
│   └── telemetry.rs        # 精度遥测
├── integration/
│   ├── mod.rs              # 集成模块根
│   ├── ruvector.rs         # RuVector 集成
│   └── ruvllm.rs           # RuvLLM 集成
└── ops.rs                  # 操作符定义
```

### 测试文件

```
tests/
├── backend_simd_tests.rs   # 后端 SIMD 测试
├── integration/
│   ├── model_loading_tests.rs
│   └── sparse_inference_tests.rs
├── unit/
│   ├── predictor_tests.rs
│   └── quantization_tests.rs
└── property/
    └── mod.rs
```

### 基准测试

```
benches/
├── sparse_inference_bench.rs
└── simd_kernels.rs
```

### 示例

```
examples/
├── basic_usage.rs
└── gguf_loader.rs
```

## 相关模块

- **ruvllm**: 本地 LLM 推理引擎
- **ruvector-attention**: 注意力机制优化
- **ruvector-learning-wasm**: WASM 微调支持
- **ruvector-solver**: 子线性求解器

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录稀疏推理引擎架构
- 完成后端和预测器接口文档
