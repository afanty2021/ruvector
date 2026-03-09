[根目录](../../CLAUDE.md) > [crates](../) > **mcp-brain-server**

# mcp-brain-server

> **π.ruv.io Shared Brain Service**
> Axum REST API 服务，提供 Firestore + GCS 后端的共享认知服务

## 模块职责

`mcp-brain-server` 是部署在 Google Cloud Run 上的 Rust 服务，为 RuVector 共享大脑提供 HTTP API。该服务集成 RuVector 认知栈（SONA、mincut、nervous-system），提供持久化存储（Firestore）、大对象存储（GCS）和实时流处理。

### 核心功能

- **HTTP REST API**: Axum 框架构建的 RESTful 接口
- **认知引擎**: 集成 SONA、mincut、nervous-system
- **持久化存储**: Firestore 数据库
- **大对象存储**: Google Cloud Storage
- **认证授权**: Ed25519 签名验证
- **速率限制**: 基于 reputation 的限流

## 入口与启动

### 服务入口

```rust
// src/main.rs
#[tokio::main]
async fn main() -> Result<()> {
    let app = create_router().await?;
    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await?;
    axum::serve(listener, app).await?;
    Ok(())
}
```

### 本地运行

```bash
# 设置环境变量
export FIRESTORE_PROJECT_ID="ruv-dev"
export GCS_BUCKET="ruv-brain-storage"
export ED25519_PRIVATE_KEY="..."

# 运行服务
cargo run -p mcp-brain-server
```

### Cloud Run 部署

```bash
# 构建 Docker 镜像
gcloud builds submit --config=cloudbuild.yaml .

# 部署到 Cloud Run
gcloud run deploy ruvbrain \
  --image gcr.io/ruv-dev/ruvbrain:latest \
  --region us-central1 \
  --project ruv-dev
```

## 对外接口

### HTTP API 端点

```rust
// GET /health - 健康检查
async fn health_check() -> Json<HealthStatus>

// POST /api/v1/embeddings - 生成嵌入
async fn create_embeddings(
    Json(req): Json<EmbeddingRequest>
) -> Result<Json<EmbeddingResponse>>

// POST /api/v1/cognitive - 认知处理
async fn cognitive_process(
    Json(req): Json<CognitiveRequest>
) -> Result<Json<CognitiveResponse>>

// POST /api/v1/graph - 图操作
async fn graph_query(
    Json(req): Json<GraphRequest>
) -> Result<Json<GraphResponse>>

// GET /api/v1/memory/{id} - 内存检索
async fn get_memory(
    Path(id): Path<String>
) -> Result<Json<MemoryResponse>>
```

### 认证

```rust
// Ed25519 签名验证
pub async fn verify_signature(
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let signature = req.headers().get("X-Signature")
        .ok_or(StatusCode::UNAUTHORIZED)?;
    let public_key = extract_public_key(&req)?;
    // 验证签名...
    Ok(next.run(req).await)
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `axum` | 0.7 | Web 框架 |
| `tokio` | 1.41 | 异步运行时 |
| `tower-http` | 0.6 | HTTP 中间件 |
| `reqwest` | 0.12 | HTTP 客户端 |
| `ed25519-dalek` | 2 | 签名验证 |
| `sona` | local | SONA 认知架构 |
| `ruvector-mincut` | local | Min-cut 算法 |
| `ruvector-nervous-system` | local | 神经系统 |
| `ruvllm` | local | LLM 推理 |

### 环境变量

```bash
# Firestore
FIRESTORE_PROJECT_ID=ruv-dev
FIRESTORE_EMULATOR_HOST=localhost:8080

# GCS
GCS_BUCKET=ruv-brain-storage
GCS_CREDENTIALS_PATH=/path/to/credentials.json

# 认证
ED25519_PRIVATE_KEY_BASE64=...
ED25519_PUBLIC_KEY_BASE64=...

# 认知栈
SONA_ENABLED=true
MINCUT_ENABLED=true
NERVOUS_SYSTEM_ENABLED=true
RUVLLM_MODEL_PATH=/models/ruvltra-medium.gguf
```

### Dockerfile

```dockerfile
FROM rust:1.77 as builder
WORKDIR /app
COPY . .
RUN cargo build --release -p mcp-brain-server

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/mcp-brain-server /usr/local/bin/
CMD ["mcp-brain-server"]
```

## 数据模型

### CognitiveRequest

```rust
pub struct CognitiveRequest {
    pub input: String,
    pub context: Option<String>,
    pub mode: CognitiveMode,
    pub options: CognitiveOptions,
}

pub enum CognitiveMode {
    Instant,      // <1ms 响应
    Background,   // ~100ms 后台处理
    Deep,         // 深度思考（分钟级）
}
```

### EmbeddingResponse

```rust
pub struct EmbeddingResponse {
    pub embedding: Vec<f32>,
    pub dimension: usize,
    pub model: String,
    pub processing_time_ms: f64,
}
```

### GraphRequest

```rust
pub struct GraphRequest {
    pub query: GraphQuery,
    pub params: GraphParams,
}

pub enum GraphQuery {
    Neighbors { node: String, direction: Direction },
    Path { from: String, to: String },
    Subgraph { nodes: Vec<String> },
}
```

## 测试与质量

### 测试文件

```
src/
└── tests.rs          # 集成测试
```

### 测试运行

```bash
# 运行测试
cargo test -p mcp-brain-server

# 运行集成测试（需要 Firestore emulator）
gcloud beta emulators firestore start
FIRESTORE_EMULATOR_HOST=localhost:8080 cargo test -p mcp-brain-server
```

### 质量指标

- 集成测试覆盖：API 端点测试
- 负载测试：使用 Locust 或 k6
- 错误处理：完整的错误类型定义

## 常见问题 (FAQ)

### Q1: 如何处理并发请求？

使用 Axum 的异步处理和 Tower 的中间件：
- `tower-http::limit::RequestBodyLimitLayer` 限制请求体大小
- 自定义 `rate_limit` 模块基于 reputation 限流
- Firestore 事务处理并发写入

### Q2: 如何监控服务健康？

提供 `/health` 端点返回：
- 服务状态
- 数据库连接状态
- 内存使用情况
- 请求处理统计

### Q3: 如何扩展服务？

Cloud Run 自动扩展：
- 设置最小/最大实例数
- 基于并发数或 CPU 使用扩展
- 使用 Firestore 分片减少热点

## 相关文件清单

### 源文件

```
src/
├── main.rs            # 服务入口
├── lib.rs             # 库接口
├── routes.rs          # HTTP 路由
├── auth.rs            # 认证中间件
├── store.rs           # Firestore 存储
├── gcs.rs             # GCS 存储
├── embeddings.rs      # 嵌入生成
├── cognitive.rs       # 认知处理
├── graph.rs           # 图查询
├── pipeline.rs        # 处理管道
├── midstream.rs       # 实时流处理
├── aggregate.rs       # 聚合操作
├── ranking.rs         # 排序算法
├── drift.rs           # 漂移检测
├── reputation.rs      # 声誉系统
├── rate_limit.rs      # 速率限制
├── verify.rs          # 签名验证
├── types.rs           # 类型定义
└── tests.rs           # 集成测试
```

### 配置文件

```
cloudbuild.yaml       # Cloud Build 配置
Dockerfile            # Docker 镜像
Cargo.toml            # 依赖配置
static/               # 静态文件
├── index.html        # π.ruv.io 首页
└── origin.html       # 原理说明页
```

## 相关模块

- **sona**: 自优化神经架构
- **ruvector-mincut**: Min-cut 算法
- **ruvector-nervous-system**: 神经系统
- **ruvllm**: LLM 推理
- **mcp-brain**: MCP 协议实现

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录 Cloud Run 服务架构
- 完成 API 端点和认证文档
