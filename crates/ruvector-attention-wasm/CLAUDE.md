[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-attention-wasm**

# ruvector-attention-wasm

> **WebAssembly Bindings for RuVector Attention**
> 46 种注意力机制的浏览器和 WASM 运行时支持

## 模块职责

`ruvector-attention-wasm` 通过 wasm-bindgen 为 `ruvector-attention` 提供 WebAssembly 绑定，使注意力机制能够在浏览器、Node.js WASM 运行时和其他支持 WASM 的环境中运行。该模块优化了 WASM 包大小，同时保持了高性能。

### 核心功能

- **WASM 绑定**: 完整的 wasm-bindgen 支持
- **46 种注意力**: 支持所有注意力机制类型
- **浏览器优化**: 针对浏览器环境优化
- **小包体积**: 优化的 WASM 打包

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/attention-wasm
```

### 基本使用

```javascript
import { FlashAttention, LinearAttention } from '@ruvector/attention-wasm';

// 创建 Flash Attention
const attention = new FlashAttention(8, 64, true);

// 准备数据
const query = new Float32Array(seqLen * hiddenDim);
const key = new Float32Array(seqLen * hiddenDim);
const value = new Float32Array(seqLen * hiddenDim);

// 计算注意力
const output = attention.forward(query, key, value);
```

### 在 React 中使用

```jsx
import { useEffect, useState } from 'react';
import { HyperbolicAttention } from '@ruvector/attention-wasm';

function AttentionComponent() {
  const [result, setResult] = useState(null);

  useEffect(() => {
    const attention = new HyperbolicAttention(8, 64);
    const output = attention.forward(q, k, v);
    setResult(output);
  }, []);

  return <div>{/* 渲染结果 */}</div>;
}
```

## 对外接口

### JavaScript API

```javascript
class FlashAttention {
  constructor(numHeads, headDim, useCausalMask = false);
  forward(query, key, value);
  backward(gradOutput);
}

class LinearAttention {
  constructor(numHeads, headDim, featureMap = 'elu');
  forward(query, key, value);
}

class HyperbolicAttention {
  constructor(numHeads, headDim, curvature = 1.0);
  forward(query, key, value);
}
```

### 辅助函数

```javascript
// 创建张量
function createTensor(shape, data);

// 释放内存
function freeTensor(tensor);

// 获取 WASM 内存
function getWasmMemory();
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-attention` | 2.0 | 核心 Rust 实现 |
| `wasm-bindgen` | 0.2 | WASM 绑定生成 |
| `js-sys` | 0.3 | JavaScript 类型绑定 |
| `web-sys` | 0.3 | Web API 绑定 |
| `serde-wasm-bindgen` | 0.6 | 序列化支持 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen = "0.2"
getrandom = { version = "0.2", features = ["js"] }

[profile.release]
opt-level = "s"
lto = true
codegen-units = 1

[package.metadata.wasm-pack.profile.release]
wasm-opt = false
```

### 特性标志

- `default`: 启用 console_error_panic_hook
- `console_error_panic_hook`: 提供更好的错误信息

## 数据模型

### 张量表示

```javascript
// TypedArray 表示
const tensor = {
  data: Float32Array,
  shape: [batchSize, seqLen, hiddenDim]
};

// 多维索引
function getIdx(tensor, batch, seq, hidden) {
  return batch * tensor.shape[1] * tensor.shape[2] +
         seq * tensor.shape[2] +
         hidden;
}
```

### 配置对象

```javascript
const config = {
  numHeads: 8,
  headDim: 64,
  useCausalMask: true,
  dropout: 0.1,
  scale: null  // 自动计算
};
```

## 测试与质量

### 测试文件

使用 `wasm-bindgen-test` 进行浏览器测试。

### 测试运行

```bash
# 构建 WASM
wasm-pack build --release --target web

# 运行测试（需要 geckodriver）
wasm-pack test --firefox --headless

# 或使用 Chrome
wasm-pack test --chrome --headless
```

### Node.js WASM 测试

```bash
# 在 Node.js 中测试
node --experimental-wasi-installed test-node.js
```

### 质量指标

- 单元测试覆盖：核心注意力计算
- WASM 包大小：<100KB（gzip）
- 性能基准：与原生 Rust 对比

## 常见问题 (FAQ)

### Q1: WASM 包大小如何优化？

使用 `opt-level = "s"` 和 LTO 减小包大小。启用 `wasm-opt` 可进一步优化，但会增加构建时间。当前配置下包大小约 80-120KB。

### Q2: 如何处理大张量？

WASM 线性内存有限制（通常 4GB）。对于非常大的张量，考虑：
1. 分块处理
2. 使用 Web Worker 并行处理
3. 利用 WebGPU 进行 GPU 加速

### Q3: 浏览器兼容性如何？

支持所有现代浏览器（Chrome、Firefox、Safari、Edge）的最新版本。需要 WASM 支持，IE 不支持。

## 相关文件清单

### 源文件

```
src/
└── lib.rs              # WASM 绑定实现
```

### 构建文件

```
Cargo.toml             # Rust 依赖配置
package.json           # NPM 包配置
pkg/                   # WASM 输出目录
├── *.wasm             # WASM 二进制
├── *.js               # JavaScript 胶水代码
└── package.json       # WASM 包配置
```

## 相关模块

- **ruvector-attention**: 核心 Rust 实现
- **ruvector-attention-node**: Node.js 原生绑定
- **ruvector-attention-unified-wasm**: 统一 WASM 绑定
- **ruvector-gnn-wasm**: GNN WASM 绑定

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 WASM 绑定架构
- 完成 JavaScript API 文档
