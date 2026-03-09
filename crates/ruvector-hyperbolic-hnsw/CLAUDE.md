[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-hyperbolic-hnsw**

# ruvector-hyperbolic-hnsw

> **Hyperbolic HNSW for Hierarchical Vector Search**
> 双曲空间嵌入与 HNSW 索引，支持层次结构感知的向量搜索

## 模块职责

`ruvector-hyperbolic-hnsw` 实现双曲空间（Poincaré 球模型）中的向量嵌入和 HNSW（层次可导航小世界图）索引。该模块特别适合表示和处理具有自然层次结构的数据（如分类法、本体论、知识图谱），在非欧几里得空间中提供高效的相似性搜索。

### 核心功能

- **Poincaré 嵌入**: 双曲空间向量表示
- **HNSW 索引**: 双曲空间中的层次索引
- **层次感知**: 利用数据层次结构优化搜索
- **相似性搜索**: 双曲距离度量
- **分片支持**: 分布式索引分片

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use poincare::{PoincareEmbedding, PoincareBall};
pub use hnsw::{HyperbolicHnsw, HnswConfig};
pub use tangent::TangentSpace;
pub use shard::HyperbolicShard;
```

### 初始化示例

```rust
use ruvector_hyperbolic_hnsw::{HyperbolicHnsw, HnswConfig};

// 创建双曲 HNSW 索引
let config = HnswConfig {
    dim: 128,
    ef_construction: 200,
    m: 16,
    curvature: 1.0,
};

let mut index = HyperbolicHnsw::new(config);

// 添加向量
index.add_vector(id, vector)?;

// 搜索最近邻
let results = index.search(query, k)?;
```

## 对外接口

### HyperbolicHnsw

```rust
pub struct HyperbolicHnsw {
    config: HnswConfig,
    embedding: PoincareEmbedding,
    hnsw: HnswIndex,
}

impl HyperbolicHnsw {
    pub fn new(config: HnswConfig) -> Self;
    pub fn add_vector(&mut self, id: u64, vec: Vec<f32>) -> Result<()>;
    pub fn search(&self, query: &[f32], k: usize) -> Result<Vec<Neighbor>>;
    pub fn save(&self, path: &str) -> Result<()>;
    pub fn load(&mut self, path: &str) -> Result<()>;
}
```

### PoincareEmbedding

```rust
pub struct PoincareEmbedding {
    dim: usize,
    curvature: f32,
}

impl PoincareEmbedding {
    pub fn new(dim: usize, curvature: f32) -> Self;
    pub fn embed(&self, vec: &[f32]) -> Result<PoincareVector>;
    pub fn distance(&self, a: &PoincareVector, b: &PoincareVector) -> f32;
    pub fn mobius_add(&self, a: &PoincareVector, b: &PoincareVector) -> PoincareVector;
    pub fn exp_map(&self, v: &TangentVector) -> PoincareVector;
    pub fn log_map(&self, p: &PoincareVector) -> TangentVector;
}
```

### PoincareVector

```rust
pub struct PoincareVector {
    coords: Vec<f32>,
    curvature: f32,
}

impl PoincareVector {
    pub fn from_euclidean(vec: Vec<f32>, curvature: f32) -> Result<Self>;
    pub fn to_euclidean(&self) -> Vec<f32>;
    pub fn norm(&self) -> f32;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `nalgebra` | 0.34.1 | 线性代数 |
| `ndarray` | 0.17.1 | 多维数组 |
| `rayon` | 1.10 | 并行处理 |
| `serde` | 1.0 | 序列化 |
| `rand` | 0.8 | 随机数 |
| `thiserror` | 2.0 | 错误处理 |

### 特性标志

- `default`: 启用 simd 和 parallel
- `simd`: SIMD 加速
- `parallel`: 多线程并行
- `wasm`: WASM 支持

### 配置参数

```rust
pub struct HnswConfig {
    pub dim: usize,
    pub ef_construction: usize,
    pub m: usize,
    pub curvature: f32,
    pub num_threads: Option<usize>,
}
```

## 数据模型

### 双曲距离

```rust
// Poincaré 球模型距离
pub fn poincare_distance(a: &PoincareVector, b: &PoincareVector) -> f32 {
    let diff = &a.coords - &b.coords;
    let norm_a = a.coords.norm();
    let norm_b = b.coords.norm();
    let norm_diff = diff.norm();

    let numerator = 2.0 * norm_diff * norm_diff;
    let denominator = (1.0 - norm_a * norm_a) * (1.0 - norm_b * norm_b);
    acosh(1.0 + numerator / denominator)
}
```

### TangentVector

```rust
pub struct TangentVector {
    coords: Vec<f32>,
    base_point: PoincareVector,
}
```

### Neighbor

```rust
pub struct Neighbor {
    pub id: u64,
    pub distance: f32,
    pub vector: PoincareVector,
}
```

## 测试与质量

### 测试文件

```
tests/
└── math_tests.rs      # 数学操作测试
```

### 基准测试

```
benches/
└── hyperbolic_bench.rs
```

### 测试运行

```bash
# 运行测试
cargo test -p ruvector-hyperbolic-hnsw

# 运行基准测试
cargo bench -p ruvector-hyperbolic-hnsw

# 带 SIMD 测试
cargo test -p ruvector-hyperbolic-hnsw --features simd
```

### 质量指标

- 单元测试覆盖：所有数学操作
- 属性测试：双曲几何性质
- 基准测试：搜索性能、索引构建

## 常见问题 (FAQ)

### Q1: 何时使用双曲空间？

双曲空间特别适合具有指数增长体积的层次结构数据，如：
- 分类法和本体论（动物 → 哺乳动物 → 犬科 → 狗）
- 知识图谱（实体层次）
- 社交网络（社区结构）
- 文档层次（章节 → 节 → 段落）

### Q2: 与欧几里得空间相比如何？

双曲空间中层次结构可以用更少维度的向量表示，且层次距离更自然。对于纯欧几里得数据，性能可能不如标准 HNSW。

### Q3: 如何选择曲率参数？

曲率控制空间的双曲程度：
- `curvature = 1.0`: 标准双曲空间
- `curvature → 0`: 接近欧几里得空间
- `curvature > 1`: 更强双曲性（层次更明显）

建议从 1.0 开始，根据数据调整。

## 相关文件清单

### 源文件

```
src/
├── lib.rs              # 库入口
├── poincare.rs         # Poincaré 嵌入
├── hnsw.rs             # HNSW 索引
├── tangent.rs          # 切空间操作
└── shard.rs            # 分片支持
```

### 测试文件

```
tests/
└── math_tests.rs       # 数学测试
```

### 基准测试

```
benches/
└── hyperbolic_bench.rs
```

## 相关模块

- **ruvector-hyperbolic-hnsw-wasm**: WASM 绑定
- **ruvector-core**: 核心 HNSW 实现
- **ruvector-graph**: 图数据库集成
- **ruvector-gnn**: 层次图神经网络

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录双曲空间嵌入架构
- 完成 Poincaré 和 HNSW 文档
