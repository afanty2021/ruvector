[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-replication**

# ruvector-replication

> **Multi-Master Data Replication for RuVector**
> 分布式数据复制与同步，支持向量时钟和冲突解决

## 模块职责

`ruvector-replication` 实现 RuVector 的多主复制功能，允许数据在多个节点间同步，同时处理冲突检测和解决。该模块使用向量时钟跟踪因果关系，并提供自动和手动的冲突解决策略。

### 核心功能

- **多主复制**: 支持多写入节点的数据复制
- **向量时钟**: 因果关系跟踪和冲突检测
- **冲突解决**: 自动和手动冲突解决策略
- **故障转移**: 节点故障时的自动转移
- **流式同步**: 高效的数据同步流

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use replica::{Replica, ReplicaConfig};
pub use conflict::{ConflictResolver, ConflictStrategy};
pub use sync::{SyncStream, SyncConfig};
pub use failover::{FailoverManager, FailoverPolicy};
```

### 初始化示例

```rust
use ruvector_replication::{Replica, ReplicaConfig, ConflictStrategy};

let config = ReplicaConfig {
    replica_id: "replica-1".to_string(),
    peers: vec![
        "replica-2".to_string(),
        "replica-3".to_string(),
    ],
    conflict_strategy: ConflictStrategy::LastWriteWins,
    sync_interval: Duration::from_secs(5),
};

let replica = Replica::new(config)?;
replica.start().await?;
```

## 对外接口

### Replica

```rust
pub struct Replica {
    id: String,
    store: Arc<RwLock<ReplicaStore>>,
    vector_clock: Arc<RwLock<VectorClock>>,
    sync_manager: SyncManager,
}

impl Replica {
    pub fn new(config: ReplicaConfig) -> Result<Self>;
    pub async fn start(&mut self) -> Result<()>;
    pub async fn write(&mut self, key: String, value: Vec<u8>) -> Result<Version>;
    pub async fn read(&self, key: &str) -> Result<Option<Value>>;
    pub async fn sync(&mut self) -> Result<SyncStats>;
    pub fn vector_clock(&self) -> VectorClock;
}
```

### ConflictResolver

```rust
pub trait ConflictResolver: Send + Sync {
    fn resolve(&self, conflicts: Vec<ConflictingValue>) -> ResolvedValue;
}

pub enum ConflictStrategy {
    LastWriteWins,
    FirstWriteWins,
    Custom(Arc<dyn ConflictResolver>),
}
```

### SyncStream

```rust
pub struct SyncStream {
    pub replica_id: String,
    pub changes: Vec<Change>,
    pub vector_clock: VectorClock,
}

pub struct Change {
    pub key: String,
    pub value: Vec<u8>,
    pub version: Version,
    pub timestamp: DateTime<Utc>,
}
```

## 关键依赖与配置

### 依赖项

| 依赖 | 版本 | 用途 |
|------|------|------|
| `ruvector-core` | 2.0.1 | 核心向量数据库 |
| `tokio` | workspace | 异步运行时 |
| `serde` | workspace | 序列化 |
| `dashmap` | workspace | 并发哈希表 |
| `parking_lot` | workspace | 高性能锁 |
| `bincode` | workspace | 二进制编码 |
| `chrono` | workspace | 时间处理 |

### 特性标志

无特定特性标志。

### 配置参数

```rust
pub struct ReplicaConfig {
    pub replica_id: String,
    pub peers: Vec<String>,
    pub conflict_strategy: ConflictStrategy,
    pub sync_interval: Duration,
    pub max_sync_retries: usize,
    pub heartbeat_interval: Duration,
}
```

## 数据模型

### VectorClock

```rust
pub struct VectorClock {
    pub entries: HashMap<String, u64>,
}

impl VectorClock {
    pub fn increment(&mut self, replica_id: &str);
    pub fn merge(&mut self, other: &VectorClock);
    pub fn compare(&self, other: &VectorClock) -> Order;
    pub fn is_concurrent(&self, other: &VectorClock) -> bool;
}
```

### Version

```rust
pub struct Version {
    pub replica_id: String,
    pub counter: u64,
    pub vector_clock: VectorClock,
    pub timestamp: DateTime<Utc>,
}
```

### ConflictingValue

```rust
pub struct ConflictingValue {
    pub key: String,
    pub versions: Vec<(Version, Vec<u8>)>,
}
```

## 测试与质量

### 测试文件

- 单元测试：嵌入在各源文件中
- 集成测试：位于 `tests/` 目录

### 测试运行

```bash
# 运行所有测试
cargo test -p ruvector-replication

# 运行特定测试
cargo test -p ruvector-replication test_vector_clock

# 测试冲突解决
cargo test -p ruvector-replication test_conflict_resolution
```

### 质量指标

- 单元测试覆盖：向量时钟操作、冲突检测
- 集成测试覆盖：多副本同步场景
- 属性测试：使用 proptest 验证一致性

## 常见问题 (FAQ)

### Q1: 如何处理并发写入冲突？

使用向量时钟检测并发更新。当检测到冲突时，根据配置的策略（LastWriteWins、FirstWriteWins 或自定义解析器）自动解决。也可以标记冲突供手动解决。

### Q2: 网络分区后如何恢复？

分区期间，每个分区的节点可以继续接受写入。分区恢复后，SyncStream 会交换所有变更，检测并解决冲突。向量时钟确保所有因果关系正确维护。

### Q3: 性能和一致性如何权衡？

可以配置同步间隔：短间隔提供更好一致性但增加网络负载。也可以使用最终一致性模型，允许短暂的不一致窗口以提高性能。

## 相关文件清单

### 源文件

```
src/
├── lib.rs           # 库入口，公开 API
├── replica.rs       # Replica 实现
├── conflict.rs      # 冲突检测和解决
├── sync.rs          # 同步流管理
├── failover.rs      # 故障转移逻辑
└── stream.rs        # 数据流处理
```

### 测试文件

```
tests/
├── integration.rs   # 集成测试
└── vector_clock.rs  # 向量时钟测试
```

### 配置文件

```
Cargo.toml           # 依赖和特性配置
README.md            # 项目文档
```

## 相关模块

- **ruvector-raft**: 强一致性共识协议
- **ruvector-cluster**: 集群成员管理
- **ruvector-delta-consensus**: CRDT 共识
- **ruvector-core**: 底层存储

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录多主复制架构
- 完成向量时钟和冲突解决文档
