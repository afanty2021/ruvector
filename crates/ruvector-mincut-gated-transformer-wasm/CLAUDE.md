[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-mincut-gated-transformer-wasm**

# ruvector-mincut-gated-transformer-wasm

> **WASM Bindings for Mincut-Gated Transformer**
| WASM 绑定，支持浏览器中的超低延迟 Transformer 推理

## 模块职责

`ruvector-mincut-gated-transformer-wasm` 通过 wasm-bindgen 为 `ruvector-mincut-gated-transformer` 提供 WebAssembly 绑定，使基于 min-cut 门控的超低延迟 Transformer 能够在浏览器和 WASM 运行时中运行。

### 核心功能

- **WASM 绑定**: 完整的门控 Transformer WASM 接口
- **浏览器优化**: 小包体积，快速加载
- **低延迟**: 优化延迟至 <1ms
- **能量门控**: 智能计算跳过

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/mincut-gated-transformer-wasm
```

### 基本使用

```javascript
import { MincutGatedTransformer } from '@ruvector/mincut-gated-transformer-wasm';

// 创建 Transformer
const transformer = new MincutGatedTransformer({
  numLayers: 6,
  hiddenDim: 512,
  numHeads: 8,
  enableSpikeAttention: true,
  gateThreshold: 0.5
});

// 前向传播
const output = await transformer.forward(input);
console.log('Output:', output);
```

### 在 React 中使用

```jsx
import { useState, useEffect } from 'react';
import { MincutGatedTransformer } from '@ruvector/mincut-gated-transformer-wasm';

function TransformerComponent() {
  const [transformer, setTransformer] = useState(null);
  const [output, setOutput] = useState(null);

  useEffect(() => {
    const tf = new MincutGatedTransformer({
      numLayers: 4,
      hiddenDim: 256,
      numHeads: 4
    });
    setTransformer(tf);
  }, []);

  const handleForward = async () => {
    if (transformer) {
      const result = await transformer.forward(input);
      setOutput(result);
    }
  };

  return (
    <div>
      <button onClick={handleForward}>Run</button>
      {output && <div>{JSON.stringify(output)}</div>}
    </div>
  );
}
```

## 对外接口

### JavaScript API

```javascript
class MincutGatedTransformer {
  constructor(config);

  // 核心方法
  async forward(input);
  async forwardWithGateInfo(input);

  // 配置
  getConfig();
  updateConfig(newConfig);

  // 查询
  getGateInfo();
  getPartitionStats();
  getMemoryUsage();

  // 生命周期
  dispose();
}
```

### 配置选项

```javascript
interface TransformerConfig {
  numLayers: number;
  hiddenDim: number;
  numHeads: number;
  intermediateDim?: number;
  attentionWindow?: number;
  slidingWindow?: number;

  // 门控配置
  gateThreshold: number;
  enableEnergyGate?: boolean;

  // 注意力配置
  enableSpikeAttention?: boolean;
  enableSpectralPE?: boolean;
  enableSparseAttention?: boolean;

  // 量化配置
  quantizationBits?: number;
}
```

### 返回类型

```javascript
interface TransformerOutput {
  data: Float32Array;
  shape: number[];
  gateInfo?: GateInfo;
}

interface GateInfo {
  totalGates: number;
  activeGates: number;
  energySaved: number;
  latencyReduction: number;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-mincut-gated-transformer` | local | 核心 Rust 实现 |
| `wasm-bindgen` | workspace | WASM 绑定 |
| `serde-wasm-bindgen` | 0.6 | 序列化 |
| `js-sys` | workspace | JS 类型 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[profile.release]
opt-level = "s"  # 平衡大小和速度
lto = true
codegen-units = 1
```

### 包大小

- 未优化 WASM: ~400KB
- gzip 压缩: ~60-100KB
- 启用特性后: +50-150KB

## 数据模型

### 张量表示

```javascript
// 使用 TypedArray 表示
class Tensor {
  constructor(data: Float32Array, shape: number[]);

  readonly data: Float32Array;
  readonly shape: number[];

  // 方法
  reshape(newShape: number[]): Tensor;
  slice(axes: number[], starts: number[], ends: number[]): Tensor;
  transpose(): Tensor;
}
```

## 测试与质量

### 测试文件

使用 `wasm-bindgen-test` 进行测试。

### 测试运行

```bash
# 构建 WASM
wasm-pack build --release --target web

# 运行测试
wasm-pack test --firefox --headless

# Node.js 测试
node test.js
```

### 质量指标

- 单元测试覆盖：核心 API
- 性能测试：延迟基准
- 内存测试：泄漏检测
- 兼容性测试：浏览器版本

## 常见问题 (FAQ)

### Q1: 与原生版本相比性能如何？

WASM 版本约为原生 Rust 的 50-70% 性能，但仍比纯 JS 快 5-20 倍。对于大多数 Web 应用，性能足够。

### Q2: 如何优化首次加载？

1. 使用异步加载：`WebAssembly.instantiateStreaming`
2. 预加载关键资源
3. 考虑使用 Service Worker 缓存
4. 延迟加载非关键特性

### Q3: 内存限制如何处理？

WASM 线性内存默认 16MB，可增长到最大 4GB。对于大模型：
1. 使用分块处理
2. 按需分配和释放
3. 监控 `getMemoryUsage()`

## 相关文件清单

### 源文件

```
src/
└── lib.rs              # WASM 绑定实现
```

### 构建文件

```
Cargo.toml             # Rust 配置
package.json           # NPM 配置
pkg/                   # WASM 输出
```

## 相关模块

- **ruvector-mincut-gated-transformer**: 核心 Rust 实现
- **ruvector-mincut**: Min-cut 算法
- **ruvector-attention-wasm**: 注意力 WASM
- **ruvector-learning-wasm**: 学习 WASM

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 WASM 绑定架构
- 完成 JavaScript API 文档
