[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-attention-node**

# ruvector-attention-node

> **Node.js Bindings for RuVector Attention**
> 46 种注意力机制的 Node.js 原生绑定

## 模块职责

`ruvector-attention-node` 通过 NAPI-RS 为 `ruvector-attention` 提供 Node.js 绑定，使 JavaScript/TypeScript 代码能够直接调用高性能的 Rust 实现的注意力机制。该模块支持 46 种注意力类型，包括 Flash Attention、线性注意力、超几何注意力等。

### 核心功能

- **原生绑定**: NAPI-RS 提供的高性能 Node.js 绑定
- **46 种注意力**: 完整支持所有注意力机制类型
- **异步支持**: 支持 Node.js 异步 API
- **跨平台**: 预编译二进制支持主流平台

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/attention-node
```

### 基本使用

```typescript
import { AttentionMechanism, FlashAttention } from '@ruvector/attention-node';

// 创建 Flash Attention 实例
const attention = new FlashAttention({
  numHeads: 8,
  headDim: 64,
  useCausalMask: true,
});

// 计算注意力
const output = attention.forward(query, key, value);
console.log(output.shape); // [batch, seqLen, hiddenDim]
```

## 对外接口

### AttentionMechanism

```typescript
abstract class AttentionMechanism {
  abstract forward(query: Tensor, key: Tensor, value: Tensor): Tensor;
  abstract backward(gradOutput: Tensor): Tensor[];

  readonly numHeads: number;
  readonly headDim: number;
  readonly useCausalMask: boolean;
}
```

### FlashAttention

```typescript
class FlashAttention extends AttentionMechanism {
  constructor(config: {
    numHeads: number;
    headDim: number;
    useCausalMask?: boolean;
    blockSize?: number;
  });

  forward(query: Tensor, key: Tensor, value: Tensor): Tensor;
}
```

### LinearAttention

```typescript
class LinearAttention extends AttentionMechanism {
  constructor(config: {
    numHeads: number;
    headDim: number;
    featureMap?: 'elu' | 'exp' | 'relu';
  });

  forward(query: Tensor, key: Tensor, value: Tensor): Tensor;
}
```

### Tensor 类

```typescript
class Tensor {
  constructor(data: Float32Array, shape: number[]);

  readonly data: Float32Array;
  readonly shape: number[];

  static from(array: number[][][]): Tensor;
  toArray(): number[][][];
  reshape(shape: number[]): Tensor;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-attention` | 2.0 | 核心 Rust 实现 |
| `napi` | 2 | Node.js API 绑定 |
| `napi-derive` | 2 | 绑定派生宏 |
| `serde` | 1.0 | 序列化 |
| `tokio` | 1 | 异步运行时 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
napi = { version = "2", default-features = false, features = ["napi9", "async", "serde-json"] }
napi-derive = "2"

[profile.release]
lto = true
opt-level = 3
codegen-units = 1
strip = true
```

### 平台支持

预编译二进制支持：
- `darwin-arm64` (Apple Silicon)
- `darwin-x64` (Intel Mac)
- `linux-arm64-gnu` (Linux ARM64)
- `linux-x64-gnu` (Linux x64)
- `linux-x64-musl` (Linux MUSL)
- `win32-x64-msvc` (Windows x64)
- `win32-arm64-msvc` (Windows ARM64)

## 数据模型

### AttentionConfig

```typescript
interface AttentionConfig {
  numHeads: number;
  headDim: number;
  useCausalMask?: boolean;
  dropout?: number;
  scale?: number;
}
```

### TensorShape

```typescript
type TensorShape = number[];

interface Tensor {
  readonly shape: TensorShape;
  readonly dtype: 'float32' | 'float16';
  readonly data: Float32Array;
}
```

## 测试与质量

### 测试文件

测试位于 `npm/` 目录下的各平台包中。

### 测试运行

```bash
# 安装依赖
npm install

# 运行测试
npm test

# 构建
npm run build

# 类型检查
npm run typecheck
```

### 质量指标

- 单元测试覆盖：所有注意力类型
- 类型安全：完整的 TypeScript 定义
- 性能测试：基准测试对比纯 JS 实现

## 常见问题 (FAQ)

### Q1: 如何处理不同精度的张量？

目前主要支持 `float32` 精度。`float16` 支持计划在未来版本中添加，需要底层 Rust 实现的支持。

### Q2: 异步 API 如何使用？

大多数注意力计算是同步的（CPU 密集型）。对于非常大的张量，可以考虑使用 Worker 线程或分块处理。

### Q3: 如何调试性能问题？

使用 Node.js 的性能分析工具：
```bash
node --prof your-script.js
node --prof-process isolate-0x*.log > profile.txt
```

## 相关文件清单

### 源文件

```
src/
└── lib.rs              # NAPI 绑定实现
```

### 构建文件

```
build.rs               # NAPI 构建脚本
Cargo.toml             # Rust 依赖配置
package.json           # NPM 包配置
```

### 平台包

```
npm/
├── darwin-arm64/
├── darwin-x64/
├── linux-arm64-gnu/
├── linux-x64-gnu/
├── linux-x64-musl/
├── win32-x64-msvc/
└── win32-arm64-msvc/
```

## 相关模块

- **ruvector-attention**: 核心 Rust 实现
- **ruvector-attention-wasm**: WASM 绑定
- **ruvector-gnn-node**: GNN Node.js 绑定
- **ruvector-node**: 核心向量数据库绑定

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 NAPI-RS 绑定架构
- 完成 TypeScript API 文档
