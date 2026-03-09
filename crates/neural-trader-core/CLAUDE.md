# neural-trader-core

[根目录](../../CLAUDE.md) > [crates](../) > **neural-trader-core**

## Module Description

Canonical market event types, ingest pipeline, and graph schema for RuVector Neural Trader, providing the foundational data structures and event processing pipeline for AI-driven algorithmic trading systems.

## Entry Points

- **Main Library**: `src/lib.rs` - Public API and event definitions
- **Event Types**: Market event definitions and schemas
- **Ingest Pipeline**: Data ingestion and processing
- **Graph Schema**: Trading graph data structures

## Architecture

### Core Components

1. **Market Event Types**
   - Order events
   - Trade events
   - Quote events
   - Market status events

2. **Ingest Pipeline**
   - Data normalization
   - Validation
   - Transformation
   - Persistence

3. **Graph Schema**
   - Market structure
   - Relationship types
   - Property definitions
   - Index configuration

## Key Dependencies

- `anyhow` - Error handling
- `serde` - Serialization (derive feature)

## Features

No feature flags - all functionality enabled by default.

## Market Event Types

### Order Events

```rust
pub struct OrderEvent {
    pub order_id: String,
    pub timestamp: DateTime<Utc>,
    pub symbol: String,
    pub side: Side,        // Buy or Sell
    pub quantity: f64,
    pub price: Option<f64>,
    pub order_type: OrderType,
    pub status: OrderStatus,
}
```

### Trade Events

```rust
pub struct TradeEvent {
    pub trade_id: String,
    pub timestamp: DateTime<Utc>,
    pub symbol: String,
    pub price: f64,
    pub quantity: f64,
    pub side: Side,
}
```

### Quote Events

```rust
pub struct QuoteEvent {
    pub timestamp: DateTime<Utc>,
    pub symbol: String,
    pub bid_price: f64,
    pub ask_price: f64,
    pub bid_size: f64,
    pub ask_size: f64,
}
```

## Graph Schema

### Market Structure

```
(Symbol) -[LISTED_ON]-> (Exchange)
(Symbol) -[HAS_SECTOR]-> (Sector)
(Symbol) -[HAS_INDUSTRY]-> (Industry)
(Symbol) -[TRADED]-> (TradeEvent)
(Symbol) -[QUOTED]-> (QuoteEvent)
(Symbol) -[ORDERED]-> (OrderEvent)
```

### Node Types

1. **Symbol**
   - ticker: String
   - name: String
   - market_cap: Float
   - shares_outstanding: Integer

2. **Exchange**
   - name: String
   - country: String
   - currency: String
   - timezone: String

3. **Sector**
   - name: String
   - description: String

4. **Industry**
   - name: String
   - sector: String

### Edge Types

1. **LISTED_ON**
   - since: DateTime
   - listing_type: String

2. **HAS_SECTOR**
   - weight: Float (sector allocation)

3. **HAS_INDUSTRY**
   - weight: Float (industry allocation)

4. **TRADED**
   - timestamp: DateTime
   - volume: Float

5. **QUOTED**
   - timestamp: DateTime
   - spread: Float

6. **ORDERED**
   - timestamp: DateTime
   - order_type: String

## Ingest Pipeline

### Stages

1. **Data Source**
   - Market data feeds
   - APIs
   - Files
   - Streams

2. **Normalization**
   - Standardize formats
   - Timezone conversion
   - Unit conversion
   - Symbol mapping

3. **Validation**
   - Data quality checks
   - Business rule validation
   - Outlier detection
   - Missing data handling

4. **Transformation**
   - Feature engineering
   - Technical indicators
   - Signal generation
   - Graph construction

5. **Persistence**
   - Time series storage
   - Graph database
   - Event log
   - Snapshot storage

## Usage Examples

```rust
use neural_trader_core::{OrderEvent, TradeEvent, IngestPipeline};

// Create events
let order = OrderEvent {
    order_id: "ORD-123".to_string(),
    timestamp: Utc::now(),
    symbol: "AAPL".to_string(),
    side: Side::Buy,
    quantity: 100.0,
    price: Some(150.0),
    order_type: OrderType::Limit,
    status: OrderStatus::Filled,
};

let trade = TradeEvent {
    trade_id: "TRD-456".to_string(),
    timestamp: Utc::now(),
    symbol: "AAPL".to_string(),
    price: 150.25,
    quantity: 100.0,
    side: Side::Buy,
};

// Ingest events
let pipeline = IngestPipeline::new()?;
pipeline.ingest_order(order)?;
pipeline.ingest_trade(trade)?;

// Query graph
let trades = pipeline.get_trades_for_symbol("AAPL")?;
```

## Related Modules

- [neural-trader-coherence](../neural-trader-coherence/CLAUDE.md) - Trading coherence
- [neural-trader-replay](../neural-trader-replay/CLAUDE.md) - Replay system
- [neural-trader-wasm](../neural-trader-wasm/CLAUDE.md) - WASM bindings
- [ruvector-graph](../ruvector-graph/CLAUDE.md) - Graph database

## Use Cases

1. **Algorithmic Trading**
   - Event-driven strategies
   - Real-time processing
   - Low-latency execution

2. **Market Analysis**
   - Graph-based analysis
   - Relationship mining
   - Pattern detection

3. **Backtesting**
   - Historical replay
   - Strategy evaluation
   - Performance metrics

4. **Risk Management**
   - Exposure tracking
   - Correlation analysis
   - Stress testing

## Changelog

### 2026-03-09
- Initial module documentation created
- Documented market event types and graph schema
- Identified ingest pipeline architecture
- Catalogued usage examples and use cases
