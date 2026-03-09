[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-mincut-wasm**

# ruvector-mincut-wasm

> **WebAssembly Bindings for Subpolynomial Min-Cut**
| WASM 绑定，支持浏览器中的子多项式时间动态最小割算法

## 模块职责

`ruvector-mincut-wasm` 通过 wasm-bindgen 为 `ruvector-mincut` 提供 WebAssembly 绑定，使世界首个子多项式时间的动态最小割算法能够在浏览器和 WASM 运行时中使用。该模块特别适合交互式网络分析、可视化和前端应用。

### 核心功能

- **WASM 绑定**: 完整的子多项式 min-cut WASM 接口
- **浏览器优化**: 小包体积，快速加载
- **动态更新**: 支持边插入/删除的增量更新
- **可视化集成**: 易于与 D3.js、Cytoscape.js 等库集成

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/mincut-wasm
```

### 基本使用

```javascript
import { DynamicMinCut } from '@ruvector/mincut-wasm';

// 创建 min-cut 实例
const mincut = new DynamicMinCut();

// 添加边
mincut.addEdge(0, 1, 1.0);
mincut.addEdge(1, 2, 2.0);
mincut.addEdge(0, 2, 3.0);

// 计算最小割
const cut = mincut.computeMinCut(0, 2);
console.log('Cut value:', cut.value);
console.log('Cut edges:', cut.edges);
```

### 与可视化库集成

```javascript
import cytoscape from 'cytoscape';
import { DynamicMinCut } from '@ruvector/mincut-wasm';

const cy = cytoscape({ /* ... */ });
const mincut = new DynamicMinCut();

// 从 Cytoscape 创建图
cy.nodes().forEach(node => {
  mincut.addNode(node.id());
});

cy.edges().forEach(edge => {
  mincut.addEdge(edge.source().id(), edge.target().id(), edge.data('weight'));
});

// 计算并高亮最小割
const cut = mincut.computeMinCut(sourceId, targetId);
cut.edges.forEach(([u, v]) => {
  cy.$(`#${u}-${v}`).addClass('highlight');
});
```

## 对外接口

### JavaScript API

```javascript
class DynamicMinCut {
  constructor();

  // 图操作
  addNode(id);
  addEdge(source, target, weight = 1.0);
  removeEdge(source, target);
  updateEdgeWeight(source, target, newWeight);

  // 算法
  computeMinCut(source, target);
  computeAllPairsMinCut();
  getMinimumSTCut();

  // 查询
  getCutValue(source, target);
  getCutEdges(source, target);
  getBridgeEdges();
  getArticulationPoints();

  // 监控
  getStats();
  resetStats();
}
```

### 返回类型

```javascript
// MinCutResult
{
  value: number,              // 割的值
  edges: Array<[string, string]>,  // 被割的边
  partition: [                // 节点分区
    string[],  // 源侧
    string[]   // 目标侧
  ],
  stats: {
    computeTimeMs: number,
    operations: number,
    cacheHits: number
  }
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-mincut` | 2.0 | 核心 Rust 实现 |
| `wasm-bindgen` | workspace | WASM 绑定 |
| `serde-wasm-bindgen` | 0.6 | 序列化 |
| `js-sys` | workspace | JS 类型 |
| `getrandom` | 0.2 (js) | 随机数 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[profile.release]
opt-level = "z"  # 最小化体积
lto = true
codegen-units = 1

[package.metadata.wasm-pack.profile.release]
wasm-opt = ["-O4"]  # 最激进优化
```

### 包大小

- 未优化 WASM: ~500KB
- gzip 压缩: ~80-120KB
- wasm-opt -O4: 再减小 20-30%

## 数据模型

### 图表示

```javascript
// 内部使用 CSR 格式存储
// 但用户无需关心内部实现
```

### 割结果

```javascript
interface MinCutResult {
  value: number;
  edges: Edge[];
  partition: Partition;
  stats: ComputationStats;
}

interface Edge {
  source: string;
  target: string;
  weight: number;
}

interface Partition {
  sourceSide: string[];
  targetSide: string[];
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

- 单元测试覆盖：核心算法
- 正确性测试：与已知结果对比
- 性能测试：大图基准
- 内存测试：泄漏检测

## 常见问题 (FAQ)

### Q1: 与原生版本相比性能如何？

WASM 版本约为原生 Rust 的 60-70% 性能，但仍比纯 JS 快 10-100 倍。对于大多数交互式应用，性能完全足够。

### Q2: 如何处理大图？

WASM 线性内存有限制（4GB）。对于超大图：
1. 使用流式处理
2. Web Worker 并行处理
3. 服务端计算 + 前端可视化

### Q3: 支持哪些浏览器？

所有支持 WASM 的现代浏览器：
- Chrome 57+
- Firefox 52+
- Safari 11+
- Edge 16+

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

- **ruvector-mincut**: 核心 Rust 实现
- **ruvector-mincut-node**: Node.js 原生绑定
- **ruvector-graph**: 图数据库
- **ruvector-graph-wasm**: 图 WASM 绑定

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 WASM 绑定架构
- 完成 JavaScript API 文档
