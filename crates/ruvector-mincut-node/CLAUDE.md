[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-mincut-node**

# ruvector-mincut-node

> **Node.js Native Bindings for Subpolynomial Min-Cut**
| Node.js 原生绑定，高性能子多项式时间最小割算法

## 模块职责

`ruvector-mincut-node` 通过 NAPI-RS 为 `ruvector-mincut` 提供 Node.js 原生绑定，使 JavaScript/TypeScript 代码能够直接调用世界首个子多项式时间的动态最小割算法。该模块支持边插入/删除的增量更新，适合网络分析、社区发现和图分区等应用。

### 核心功能

- **原生绑定**: NAPI-RS 高性能绑定
- **增量更新**: O(log n) 到 O(√n) 的动态更新
- **全对计算**: 高效的全对最小割
- **监控支持**: 性能监控和统计

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/mincut-node
```

### 基本使用

```typescript
import { DynamicMinCut } from '@ruvector/mincut-node';

// 创建实例
const mincut = new DynamicMinCut({
  enableMonitoring: true,
  cacheSize: 1000
});

// 构建图
mincut.addNode('A');
mincut.addNode('B');
mincut.addNode('C');
mincut.addEdge('A', 'B', 1.0);
mincut.addEdge('B', 'C', 2.0);
mincut.addEdge('A', 'C', 3.0);

// 计算最小割
const result = mincut.computeMinCut('A', 'C');
console.log(`Cut value: ${result.value}`);
console.log(`Cut edges: ${result.edges}`);
```

### 异步 API

```typescript
// 对于大图，使用异步 API
async function processLargeGraph() {
  const mincut = new DynamicMinCut();

  // 批量添加边
  await mincut.addEdgesAsync(edgeList);

  // 异步计算
  const result = await mincut.computeMinCutAsync(source, target);
  return result;
}
```

## 对外接口

### DynamicMinCut 类

```typescript
class DynamicMinCut {
  constructor(config?: MinCutConfig);

  // 图操作
  addNode(id: string): void;
  addEdge(source: string, target: string, weight?: number): void;
  removeEdge(source: string, target: string): void;
  updateEdgeWeight(source: string, target: string, weight: number): void;
  addEdges(edges: Array<[source, target, weight?]>): void;
  addEdgesAsync(edges: Array<[source, target, weight?]>): Promise<void>;

  // 算法
  computeMinCut(source: string, target: string): MinCutResult;
  computeMinCutAsync(source: string, target: string): Promise<MinCutResult>;
  computeAllPairsMinCut(): Map<string, Map<string, number>>;
  getMinimumSTCut(): STCut;

  // 查询
  getCutValue(source: string, target: string): number;
  getCutEdges(source: string, target: string): Array<[string, string]>;
  getBridgeEdges(): Array<[string, string]>;
  getArticulationPoints(): string[];

  // 监控
  getStats(): MinCutStats;
  resetStats(): void;

  // 序列化
  serialize(): Buffer;
  deserialize(buffer: Buffer): void;
}
```

### 类型定义

```typescript
interface MinCutConfig {
  enableMonitoring?: boolean;
  cacheSize?: number;
  parallelThreshold?: number;
}

interface MinCutResult {
  value: number;
  edges: Array<[string, string]>;
  partition: Partition;
  stats: ComputationStats;
}

interface Partition {
  sourceSide: string[];
  targetSide: string[];
}

interface ComputationStats {
  computeTimeMs: number;
  operations: number;
  cacheHits: number;
  algorithmType: 'jtree' | 'snn' | 'hybrid';
}

interface MinCutStats {
  totalOperations: number;
  averageUpdateTime: number;
  cacheHitRate: number;
  memoryUsage: number;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-mincut` | 2.0 | 核心 Rust 实现 |
| `napi` | workspace | Node.js API |
| `napi-derive` | workspace | 绑定派生宏 |
| `serde` | workspace | 序列化 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
napi = { workspace = true }
napi-derive = { workspace = true }

[profile.release]
lto = true
opt-level = 3
codegen-units = 1
strip = true
```

### 平台支持

预编译二进制支持：
- `darwin-arm64` (Apple Silicon M1/M2/M3/M4)
- `darwin-x64` (Intel Mac)
- `linux-arm64-gnu`
- `linux-x64-gnu`
- `linux-x64-musl`
- `win32-x64-msvc`
- `win32-arm64-msvc`

## 数据模型

### 图内部表示

```typescript
// 用户无需关心，内部使用 CSR 格式
// 节点 ID 自动映射到连续索引
```

### 序列化格式

```typescript
// 序列化的图数据
interface SerializedGraph {
  nodes: string[];
  edges: Array<[number, number, number]>;
  metadata: {
    version: number;
    timestamp: number;
  };
}
```

## 测试与质量

### 测试文件

测试位于包的 `tests/` 目录。

### 测试运行

```bash
# 安装依赖
npm install

# 运行测试
npm test

# 性能基准
npm run bench

# 类型检查
npm run typecheck
```

### 质量指标

- 单元测试覆盖：所有 API 方法
- 性能测试：与纯 JS 实现对比
- 内存测试：泄漏检测
- 正确性测试：与已知结果验证

## 常见问题 (FAQ)

### Q1: 如何选择算法？

自动选择：
- 小图（<1000 节点）：J-Tree 算法
- 中图（1000-10000）：混合算法
- 大图（>10000）：SNN 算法

可手动通过 `computeMinCut` 的选项指定。

### Q2: 内存使用如何？

内存使用与边数成正比。典型配置：
- 1000 节点，5000 边：~1MB
- 10000 节点，50000 边：~10MB
- 100000 节点，500000 边：~100MB

### Q3: 如何处理并行计算？

使用 Worker 线程并行处理：
```typescript
import { Worker } from 'worker_threads';

const worker = new Worker('./mincut-worker.js');
worker.postMessage({ graphData, source, target });
worker.on('message', (result) => {
  console.log('Min-cut result:', result);
});
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
Cargo.toml             # Rust 配置
package.json           # NPM 配置
npm/                   # 平台特定包
├── darwin-arm64/
├── darwin-x64/
├── linux-x64-gnu/
└── win32-x64-msvc/
```

## 相关模块

- **ruvector-mincut**: 核心 Rust 实现
- **ruvector-mincut-wasm**: WASM 绑定
- **ruvector-graph**: 图数据库
- **ruvector-graph-node**: 图 Node.js 绑定

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 NAPI-RS 绑定架构
- 完成 TypeScript API 文档
