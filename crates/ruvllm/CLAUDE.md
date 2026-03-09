[根目录](../../CLAUDE.md) > [crates](../) > **ruvllm**

# ruvllm

> **Local LLM Inference Engine with SONA Learning**
| 本地 LLM 推理引擎，支持 Metal、CUDA、WebGPU，集成 SONA 自适应学习

## 模块职责

`ruvllm` 是 RuVector 的本地 LLM 推理引擎，支持 GGUF 模型加载、多后端推理（Candle/Metal/CUDA/CoreML）、SONA 三层自适应学习、MicroLoRA 实时微调、HuggingFace Hub 集成等。该引擎特别适合隐私敏感、离线部署和成本优化的场景。

### 核心功能

- **多后端推理**: Candle（Metal/CUDA/CPU）、CoreML（ANE）、mistral-rs
- **SONA 学习**: 三层自适应（即时<1ms、后台~100ms、深度分钟级）
- **MicroLoRA**: 每请求微调，rank 1-2 适配器
- **GGUF 支持**: 内存映射加载，量化支持（Q4K/Q8/FP16）
- **双缓存 KV Cache**: 尾部 FP16 + 存储 Q4，自动迁移
- **连续批处理**: 动态批调度，2-3x 吞吐提升
- **投机解码**: Draft model 加速 2-3x
- **HuggingFace Hub**: 模型下载上传
- **任务适配器**: 5 个预训练 LoRA（coder/researcher/security/architect/reviewer）

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use backends::{
    candle_backend::CandleBackend,
    coreml_backend::CoreMLBackend,
    hybrid_pipeline::HybridPipeline,
    mistral_backend::MistralBackend,
};
pub use gguf::{GgufLoader, GgufConfig};
pub use lora::{MicroLoRA, LoraAdapter};
pub use optimization::{SonaLlm, SonaLlmConfig};
pub use context::{ContextManager, SemanticCache};
pub use hub::{ModelDownloader, ModelUploader, RuvLtraRegistry};
```

### 初始化示例

```rust
use ruvllm::prelude::*;

// Candle 后端（Apple Metal）
let mut backend = CandleBackend::with_device(DeviceType::Metal)?;
backend.load_gguf("models/qwen2.5-7b-q4_k.gguf", ModelConfig::default())?;

// 基础推理
let response = backend.generate("Explain quantum computing",
    GenerateParams {
        max_tokens: 256,
        temperature: 0.7,
        ..Default::default()
    }
)?;

// SONA 学习
let sona = SonaLlm::new(SonaLlmConfig::default());
sona.instant_adapt("query", "response", 0.85)?;

// 微调推理
let adapter = MicroLoRA::new(MicroLoraConfig::for_hidden_dim(4096));
backend.set_adapter(Some(adapter))?;
```

## 对外接口

### CandleBackend

```rust
pub struct CandleBackend {
    device: Device,
    model: Option<Box<dyn Model>>,
    cache: Option<TwoTierKvCache>,
    adapter: Option<MicroLoRA>,
}

impl CandleBackend {
    pub fn with_device(device_type: DeviceType) -> Result<Self>;
    pub fn load_gguf(&mut self, path: &str, config: ModelConfig) -> Result<()>;
    pub fn generate(&mut self, prompt: &str, params: GenerateParams) -> Result<String>;
    pub fn generate_stream(&mut self, prompt: &str, params: GenerateParams) -> Result<ResponseStream>;
    pub fn set_adapter(&mut self, adapter: Option<MicroLoRA>) -> Result<()>;
}
```

### CoreMLBackend

```rust
pub struct CoreMLBackend {
    ane_strategy: AneStrategy,
    model: Option<CoreMLModel>,
}

pub enum AneStrategy {
    PreferAneForMlp,      // MLP 优先用 ANE
    PreferAneForSmall,    // 小维度优先用 ANE
    Adaptive,             // 自适应选择
    GpuOnly,              // 仅用 GPU
}
```

### HybridPipeline

```rust
pub struct HybridPipeline {
    ane_strategy: AneStrategy,
    gpu_for_attention: bool,
    ane_for_mlp: bool,
}
```

### SonaLlm

```rust
pub struct SonaLlm {
    config: SonaLlmConfig,
    instant_adapter: MicroLoRA,
    background_samples: VecDeque<Sample>,
}

impl SonaLlm {
    pub fn new(config: SonaLlmConfig) -> Self;
    pub fn instant_adapt(&mut self, query: &str, response: &str, quality: f32) -> Result<AdaptResult>;
    pub fn maybe_background(&mut self) -> Result<BackgroundResult>;
    pub fn should_trigger_deep(&self) -> bool;
    pub fn deep_optimize(&mut self, trigger: OptimizationTrigger) -> Result<DeepResult>;
}
```

### ModelDownloader

```rust
pub struct ModelDownloader {
    config: DownloadConfig,
}

impl ModelDownloader {
    pub fn new(config: DownloadConfig) -> Self;
    pub fn download(&self, model_id: &str, target: Option<&str>) -> Result<PathBuf>;
    pub fn download_with_progress(&self, model_id: &str, target: Option<&str>) -> Result<ProgressStream>;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-core` | 2.0 | 向量数据库集成 |
| `ruvector-sona` | 0.1.6 | SONA 学习 |
| `candle-core` | 0.8 | Candle ML 框架 |
| `candle-transformers` | 0.8 | Transformer 模型 |
| `tokenizers` | 0.20 | 分词器 |
| `hf-hub` | 0.3 | HuggingFace Hub |
| `metal` | 0.29 | Metal GPU |
| `objc2-core-ml` | 0.3 | CoreML ANE |
| `rayon` | 1.10 | 并行处理 |
| `half` | 2.4 | FP16 |

### 特性标志

| 特性 | 描述 |
|------|------|
| `default` | async-runtime + candle |
| `async-runtime` | Tokio 异步支持 |
| `candle` | Candle 后端 |
| `metal` | Apple Metal 加速 |
| `cuda` | NVIDIA CUDA 加速 |
| `coreml` | Apple Neural Engine |
| `hybrid-ane` | GPU+ANE 混合 |
| `inference-metal` | 完整 Metal 推理栈 |
| `inference-cuda` | 完整 CUDA 推理栈 |
| `parallel` | Rayon 并行 |
| `gguf-mmap` | GGUF 内存映射 |
| `wasm` | WebAssembly 支持 |

### 环境变量

```bash
RUVLLM_CACHE_DIR=~/.cache/ruvllm
RUVLLM_LOG_LEVEL=info
RUVLLM_METAL_DEVICE=0
RUVLLM_ANE_ENABLED=true
RUVLLM_SONA_ENABLED=true
HF_TOKEN=your_token_here
```

## 数据模型

### ModelConfig

```rust
pub struct ModelConfig {
    pub max_context: usize,
    pub use_flash_attention: bool,
    pub quantization: Quantization,
    pub kv_cache_config: KvCacheConfig,
    pub rope_scaling: Option<RopeScaling>,
    pub sliding_window: Option<usize>,
}
```

### GenerateParams

```rust
pub struct GenerateParams {
    pub max_tokens: usize,
    pub temperature: f32,
    pub top_p: f32,
    pub top_k: usize,
    pub repeat_penalty: f32,
    pub stop_tokens: Vec<String>,
}
```

### MicroLoraConfig

```rust
pub struct MicroLoraConfig {
    pub rank: usize,
    pub alpha: f32,
    pub dropout: f32,
    pub target_modules: Vec<String>,
}
```

### SonaLlmConfig

```rust
pub struct SonaLlmConfig {
    pub instant_lr: f32,
    pub background_interval_ms: u64,
    pub deep_trigger_threshold: f32,
    pub consolidation_strategy: ConsolidationStrategy,
}
```

## 测试与质量

### 基准测试

```
benches/
├── attention_bench.rs
├── rope_bench.rs
├── norm_bench.rs
├── matmul_bench.rs
├── lora_bench.rs
├── e2e_bench.rs
├── metal_bench.rs
├── serving_bench.rs
├── ane_bench.rs
└── ruvltra_benchmark.rs
```

### 测试

```
tests/
└── real_model_test.rs
```

### 测试运行

```bash
# 运行测试
cargo test -p ruvllm

# 运行基准测试
cargo bench -p ruvllm --features inference-metal

# 真实模型测试
cargo test -p ruvllm --test real_model_test --features async-runtime
```

### 质量指标

- 推理基准：Qwen2.5-7B Q4K，2800 tok/s prefill，95 tok/s decode
- ANE 基准：<512 维度快 30-50%
- SONA 学习：<1ms 即时适应
- MicroLoRA：<100μs 应用延迟

## 常见问题 (FAQ)

### Q1: 如何选择后端？

| 场景 | 推荐后端 |
|------|----------|
| 单用户/边缘 | Candle |
| Apple Silicon | Candle + Metal |
| NVIDIA GPU | Candle + CUDA |
| 最大效率（Mac）| Hybrid Pipeline (GPU+ANE) |
| 生产服务（10+用户）| mistral-rs（待发布）|

### Q2: ANE vs GPU 如何选择？

| 维度 | ANE | GPU |
|------|-----|-----|
| <512 维 | +30-50% | - |
| 512-1024 | +10-30% | - |
| 1024-1536 | ~Similar | ~Similar |
| >2048 | - | +30-50% |

使用 `HybridPipeline` 自动选择最优。

### Q3: SONA 如何工作？

三层学习循环：
1. **即时层**（<1ms）：每请求 MicroLoRA 适应
2. **后台层**（~100ms）：定期模式整合
3. **深度层**（分钟）：触发完整优化

## 相关文件清单

### 源文件

```
src/
├── lib.rs                        # 库入口
├── error.rs                      # 错误类型
├── capabilities.rs               # 能力检测
├── autodetect.rs                 # 自动检测
├── adapter_manager.rs            # 适配器管理
├── kv_cache.rs                   # 双缓存 KV
├── paged_attention.rs            # 分页注意力
├── memory_pool.rs                # 内存池
├── policy_store.rs               # 策略存储
├── intelligence/                 # 智能模块
├── backends/
│   ├── mod.rs
│   ├── candle_backend.rs         # Candle 后端
│   ├── coreml_backend.rs         # CoreML 后端
│   ├── hybrid_pipeline.rs        # 混合管道
│   ├── mistral_backend.rs        # mistral-rs 后端
│   ├── gemma2.rs                 # Gemma-2 模型
│   ├── phi3.rs                   # Phi-3 模型
│   └── metal/                    # Metal 操作
├── bitnet/                       # BitNet 支持
│   ├── backend.rs
│   ├── quantizer.rs
│   ├── rlm_embedder.rs
│   └── ...
├── claude_flow/                  # Claude Flow 集成
│   ├── hnsw_router.rs
│   ├── agent_router.rs
│   └── ...
├── context/                      # 上下文管理
│   ├── context_manager.rs
│   ├── semantic_cache.rs
│   ├── episodic_memory.rs
│   └── ...
├── lora/                         # LoRA 适配器
│   ├── micro_lora.rs
│   ├── adapter.rs
│   ├── training.rs
│   └── adapters/                 # 任务适配器
├── optimization/                 # SONA 优化
│   ├── sona_llm.rs
│   ├── realtime.rs
│   └── metrics.rs
├── gguf/                         # GGUF 加载
│   ├── loader.rs
│   ├── parser.rs
│   ├── tensors.rs
│   └── ...
├── hub/                          # HuggingFace Hub
│   ├── download.rs
│   ├── upload.rs
│   └── registry.rs
├── evaluation/                   # 评估工具
│   ├── harness.rs
│   ├── real_harness.rs
│   ├── swe_bench.rs
│   └── ...
├── models/                       # 模型定义
│   ├── ruvltra.rs                # RuvLTRA 模型
│   ├── ruvltra_medium.rs         # RuvLTRA-Medium
│   └── ...
└── kernels/                      # 计算内核
    ├── attention.rs
    ├── matmul.rs
    ├── rope.rs
    ├── norm.rs
    ├── ane_ops.rs
    └── ...
```

### 基准测试

```
benches/
├── attention_bench.rs
├── rope_bench.rs
├── norm_bench.rs
├── matmul_bench.rs
├── lora_bench.rs
├── e2e_bench.rs
├── metal_bench.rs
├── serving_bench.rs
├── ane_bench.rs
└── ruvltra_benchmark.rs
```

### 示例

```
examples/
├── download_test_model.rs
├── hub_cli.rs
├── benchmark_model.rs
└── run_eval.rs
```

## 相关模块

- **ruvector-core**: 向量数据库
- **ruvector-sona**: SONA 自适应学习
- **ruvector-attention**: 注意力机制
- **ruvector-gnn**: 图神经网络
- **ruvector-graph**: 图数据库

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录多后端推理架构
- 完成 SONA、MicroLoRA、Hub 集成文档
- 添加性能基准和配置指南
