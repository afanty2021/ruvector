[根目录](../../CLAUDE.md) > [crates](../) > **ruvector-raft**

# ruvector-raft

> **Raft Consensus Implementation for RuVector**
> Distributed metadata coordination with strong consistency guarantees

## 模块职责

`ruvector-raft` 实现 Raft 共识算法，为 RuVector 分布式系统提供强一致性保证的元数据协调服务。该模块负责处理领导者选举、日志复制和状态机管理，确保在分布式环境下的数据一致性和可用性。

### 核心功能

- **领导者选举**: 基于随机超时的领导者选举机制
- **日志复制**: 日志条目复制和一致性检查
- **状态机管理**: 应用状态机并保证线性一致性
- **RPC 通信**: 节点间通信协议
- **快照机制**: 日志压缩和状态快照

## 入口与启动

### 库入口

```rust
// src/lib.rs
pub use node::RaftNode;
pub use log::{LogEntry, Command};
pub use state::{NodeState, Role};
pub use election::ElectionTimer;
pub use rpc::{RpcHandler, Message};
```

### 初始化示例

```rust
use ruvector_raft::{RaftNode, NodeConfig, Command};
use tokio::runtime::Runtime;

// 创建 Raft 节点
let config = NodeConfig {
    node_id: "node-1".to_string(),
    peers: vec!["node-2".to_string(), "node-3".to_string()],
    election_timeout: 1500,
    heartbeat_interval: 500,
};

let mut node = RaftNode::new(config);
Runtime::new()?.block_on(node.start());
```

## 对外接口

### RaftNode

```rust
pub struct RaftNode {
    id: String,
    state: Arc<RwLock<NodeState>>,
    log: Arc<RwLock<Log>>,
    election_timer: ElectionTimer,
}

impl RaftNode {
    pub fn new(config: NodeConfig) -> Self;
    pub async fn start(&mut self) -> Result<()>;
    pub async fn propose(&self, cmd: Command) -> Result<()>;
    pub async fn step_down(&mut self) -> Result<()>;
    pub fn state(&self) -> NodeState;
}
```

### Command 类型

```rust
pub enum Command {
    AddNode { id: String },
    RemoveNode { id: String },
    UpdateMetadata { key: String, value: Vec<u8> },
    Barrier,
}
```

### RPC 消息

```rust
pub enum Message {
    RequestVote {
        term: u64,
        candidate_id: String,
        last_log_index: u64,
        last_log_term: u64,
    },
    AppendEntries {
        term: u64,
        leader_id: String,
        entries: Vec<LogEntry>,
    },
    // ... 更多消息类型
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

### 特性标志

无特定特性标志。

### 配置参数

```rust
pub struct NodeConfig {
    pub node_id: String,
    pub peers: Vec<String>,
    pub election_timeout: u64,
    pub heartbeat_interval: u64,
    pub max_log_entries: usize,
    pub snapshot_threshold: usize,
}
```

## 数据模型

### LogEntry

```rust
pub struct LogEntry {
    pub index: u64,
    pub term: u64,
    pub command: Command,
    pub timestamp: DateTime<Utc>,
}
```

### NodeState

```rust
pub struct NodeState {
    pub role: Role,
    pub current_term: u64,
    pub voted_for: Option<String>,
    pub commit_index: u64,
    pub last_applied: u64,
}

pub enum Role {
    Follower,
    Candidate,
    Leader,
}
```

## 测试与质量

### 测试文件

- 集成测试：位于 `tests/` 目录
- 单元测试：嵌入在各源文件中

### 测试运行

```bash
# 运行所有测试
cargo test -p ruvector-raft

# 运行特定测试
cargo test -p ruvector-raft test_election

# 带输出的测试
cargo test -p ruvector-raft -- --nocapture
```

### 质量指标

- 单元测试覆盖：核心 Raft 算法逻辑
- 集成测试覆盖：多节点协调场景
- 属性测试：使用 proptest 验证不变性

## 常见问题 (FAQ)

### Q1: 如何处理网络分区？

Raft 通过选举超时机制处理网络分区。分区后少数派无法选举出新领导者，多数派可以继续提供服务。分区恢复后，旧领导者会通过 AppendEntries 消息发现更高的任期号并自动步下。

### Q2: 日志压缩如何工作？

当日志条目数超过 `snapshot_threshold` 时，节点会创建快照并截断已应用的日志。快照包含完整的应用状态和最后一条日志的索引/任期。

### Q3: 如何添加新节点？

通过 `Command::AddNode` 动态添加新节点。新节点先以学习者（learner）模式加入，不参与投票，待日志同步完成后再转为正式成员。

## 相关文件清单

### 源文件

```
src/
├── lib.rs           # 库入口，公开 API
├── node.rs          # RaftNode 实现
├── log.rs           # 日志管理
├── state.rs         # 节点状态和角色转换
├── election.rs      # 领导者选举逻辑
└── rpc.rs           # RPC 消息处理
```

### 测试文件

```
tests/
├── integration.rs   # 集成测试
└── property.rs      # 属性测试
```

### 配置文件

```
Cargo.toml           # 依赖和特性配置
README.md            # 项目文档
```

## 相关模块

- **ruvector-replication**: 使用 Raft 进行复制协调
- **ruvector-cluster**: 集群成员管理和发现
- **ruvector-delta-consensus**: CRDT 共识机制
- **ruvector-core**: 底层存储引擎

## 变更记录 (Changelog)

### 2026-03-09
- 初始文档创建
- 记录核心 Raft 实现结构
- 完成接口和数据模型文档
