[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-router-wasm**

# ruvector-router-wasm

> **WebAssembly Bindings for Neural Routing**
| 神经路由推理引擎的 WASM 绑定

## 模块职责

`ruvector-router-wasm` 通过 wasm-bindgen 为 `ruvector-router-core` 提供 WebAssembly 绑定，使神经路由推理引擎能够在浏览器和 WASM 运行时中使用。该模块支持智能请求路由、模型选择和负载均衡。

### 核心功能

- **WASM 绑定**: 完整的路由引擎 WASM 接口
- **神经路由**: 基于学习到的模式路由请求
- **模型选择**: 自动选择最佳模型
- **负载均衡**: 智能请求分发

## 入口与启动

### NPM 包安装

```bash
npm install @ruvector/router-wasm
```

### 基本使用

```javascript
import { NeuralRouter, RouterConfig } from '@ruvector/router-wasm';

// 创建路由器
const config = new RouterConfig({
  numModels: 5,
  embeddingDim: 512,
  cacheSize: 1000
});

const router = new NeuralRouter(config);

// 添加模型
router.addModel('model-1', { capabilities: ['code', 'analysis'] });
router.addModel('model-2', { capabilities: ['chat', 'creative'] });

// 路由请求
const result = router.route('Write a function to sort an array');
console.log('Selected model:', result.modelId);
console.log('Confidence:', result.confidence);
```

## 对外接口

### NeuralRouter

```javascript
class NeuralRouter {
  constructor(config);

  // 模型管理
  addModel(id: string, metadata: ModelMetadata): void;
  removeModel(id: string): void;
  updateModel(id: string, metadata: ModelMetadata): void;

  // 路由
  route(query: string, options?: RouteOptions): RouteResult;
  routeBatch(queries: string[], options?: RouteOptions): RouteResult[];

  // 学习
  updateFeedback(routeId: string, feedback: Feedback): void;
  getModelStats(modelId: string): ModelStats;

  // 配置
  getConfig(): RouterConfig;
  updateConfig(config: Partial<RouterConfig>): void;

  // 持久化
  save(): Uint8Array;
  load(data: Uint8Array): void;

  // 生命周期
  dispose(): void;
}
```

### RouterConfig

```javascript
class RouterConfig {
  constructor({
    numModels: number,
    embeddingDim: number,
    cacheSize: number,

    // 路由策略
    routingStrategy: 'neural' | 'semantic' | 'random',

    // 性能
    maxConcurrency: number,
    timeoutMs: number,

    // 学习
    learningRate: number,
    explorationRate: number
  });
}
```

### RouteResult

```javascript
interface RouteResult {
  modelId: string;
  confidence: number;
  latency: number;
  reasoning: RouteReasoning;
}

interface RouteReasoning {
  capabilities: string[];
  similarity: number;
  load: number;
  estimatedCost: number;
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-router-core` | 2.0 | 核心 Rust 实现 |
| `wasm-bindgen` | workspace | WASM 绑定 |
| `wasm-bindgen-futures` | 0.4 | 异步支持 |
| `serde-wasm-bindgen` | 0.6 | 序列化 |

### 构建配置

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[profile.release]
opt-level = "z"  # 最小化体积
lto = true
codegen-units = 1
panic = "abort"
strip = true
```

### 包大小

- 基础 WASM: ~100KB
- gzip 压缩: ~30-50KB

## 数据模型

### ModelMetadata

```javascript
interface ModelMetadata {
  capabilities: string[];
  maxTokens: number;
  costPerToken: number;
  avgLatency: number;
  priority: number;
}
```

### Feedback

```javascript
interface Feedback {
  success: boolean;
  latency: number;
  quality: number;
  cost: number;
  userRating?: number;
}
```

### ModelStats

```javascript
interface ModelStats {
  totalRequests: number;
  successfulRequests: number;
  avgLatency: number;
  avgQuality: number;
  totalCost: number;
  capabilities: string[];
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

- 单元测试覆盖：所有 API 方法
- 集成测试：端到端路由流程
- 性能测试：路由延迟基准
- 学习测试：反馈循环验证

## 常见问题 (FAQ)

### Q1: 路由决策如何做出？

路由器考虑多个因素：
1. **语义相似度**: 查询与模型能力的匹配
2. **当前负载**: 模型的并发请求数
3. **历史性能**: 平均延迟和质量
4. **成本**: token 计费成本
5. **探索**: 随机探索新模型

### Q2: 如何提高路由准确性？

1. 提供准确的反馈（使用 `updateFeedback`）
2. 定义清晰的模型能力
3. 调整学习率和探索率
4. 使用批量路由获取更多信息

### Q3: 如何处理模型故障？

路由器会：
1. 自动检测失败请求
2. 临时降低故障模型优先级
3. 重试到备用模型
4. 持续失败后移除模型

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

- **ruvector-router-core**: 核心 Rust 实现
- **ruvector-router-cli**: CLI 工具
- **ruvector-router-ffi**: FFI 绑定
- **ruvector-attention**: 注意力机制
- **ruvector-gnn**: 图神经网络

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录神经路由 WASM 架构
- 完成 JavaScript API 文档
