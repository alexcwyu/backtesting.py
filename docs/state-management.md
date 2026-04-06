# Backtesting.py -- State Management

## Order State Machine

```mermaid
stateDiagram-v2
    [*] --> Pending: Strategy.buy() / Strategy.sell()

    Pending --> Filled: Market order<br/>next bar open / current close
    Pending --> Filled: Limit hit<br/>low <= limit (long)
    Pending --> Filled: Stop triggered<br/>then market/limit fill
    Pending --> Canceled: Insufficient margin
    Pending --> Canceled: order.cancel()
    Pending --> Canceled: exclusive_orders=True<br/>new order placed
    Pending --> Pending: Conditions not met<br/>remains in queue (GTC)

    Filled --> TradeOpen: Creates Trade object
    TradeOpen --> HasSLTP: SL/TP specified on order

    HasSLTP --> SLOrder: Contingent stop-loss order created
    HasSLTP --> TPOrder: Contingent take-profit order created

    SLOrder --> TradeClosed: Stop-loss hit
    TPOrder --> TradeClosed: Take-profit hit
    TradeOpen --> TradeClosed: trade.close()
    TradeOpen --> TradeClosed: position.close()
    TradeOpen --> TradeClosed: FIFO close by opposite order

    TradeClosed --> [*]: Trade moved to closed_trades

    Canceled --> [*]
```

### Order Types and Fill Logic

| Order Type | Created By | Fill Condition | Fill Price |
|-----------|-----------|----------------|------------|
| **Market** | `buy()` / `sell()` with no limit/stop | Immediate (next bar) | Next open or current close |
| **Limit** | `buy(limit=X)` | `Low <= X` (long) or `High >= X` (short) | Limit price |
| **Stop-Market** | `buy(stop=X)` | `High >= X` (long) or `Low <= X` (short) | Max(open, stop) long / Min(open, stop) short |
| **Stop-Limit** | `buy(stop=X, limit=Y)` | Stop hit first, then limit condition | Min(stop, limit) long / Max(stop, limit) short |
| **SL (contingent)** | Auto on `buy(sl=X)` | Stop-market on parent trade | Stop price |
| **TP (contingent)** | Auto on `buy(tp=X)` | Limit on parent trade | Limit price |

## Position Tracking

```mermaid
flowchart TD
    subgraph Position State
        POS[Position]
        POS --> SIZE[size: sum of trade sizes]
        POS --> PL[pl: sum of trade P&L]
        POS --> PLPCT[pl_pct: weighted P&L %]
        POS --> ISLONG[is_long: size > 0]
        POS --> ISSHORT[is_short: size < 0]
    end

    subgraph Trade List
        T1[Trade 1: size=100, entry=50.0]
        T2[Trade 2: size=50, entry=52.0]
        T3[Trade 3: size=-30, entry=55.0]
    end

    T1 --> POS
    T2 --> POS
    T3 --> POS

    subgraph Example
        EX["Position.size = 100 + 50 + (-30) = 120<br/>Position.is_long = True"]
    end
```

### Position vs Trade Model

Backtesting.py distinguishes between individual `Trade` objects and the aggregate `Position`:

- **Trade**: A single fill with entry price, entry bar, optional SL/TP, and P&L tracking. Trades are stored in `broker.trades` (active) or `broker.closed_trades` (settled).
- **Position**: A read-only aggregate view computed from all active trades. `position.size` is the sum of all trade sizes. `position.close()` closes all active trades.

With `hedging=True`, long and short trades can coexist. With `hedging=False` (default), new opposite-facing orders first close existing trades via FIFO.

## Portfolio / Equity State

```mermaid
flowchart LR
    subgraph Broker State
        CASH[_cash: float<br/>Available cash]
        EQ[equity: cash + unrealized PL]
        MA[margin_available:<br/>equity - margin_used]
        EQCURVE[_equity array<br/>logged each bar]
    end

    subgraph Per-Bar Update
        OPEN[Process orders] --> FILL[Fill orders<br/>adjust cash]
        FILL --> LOG[equity_i = cash + sum trade.pl]
        LOG --> CHECK{equity <= 0?}
        CHECK -->|Yes| BANKRUPT[Close all trades<br/>equity = 0<br/>raise _OutOfMoneyError]
        CHECK -->|No| CONTINUE[Continue simulation]
    end
```

### Equity Tracking

The broker maintains an equity array (`_equity`) indexed by bar number:
- At each bar, `equity[i] = cash + sum(trade.pl for active trades)`
- If equity drops to zero or below, all trades are force-closed and the simulation stops
- The equity curve is used to compute drawdown, returns, and all risk metrics

### Cash Flow

```
Initial Cash
  - commission on trade open (deducted from cash at open time)
  + trade P&L - commission on trade close (added to cash at close time)
  = Current Cash
```

Commission is applied twice per trade: once at entry and once at exit. The entry commission is calculated using the final trade size (which may differ from the original order size due to `_reduce_trade()`).

## Strategy Lifecycle States

```mermaid
stateDiagram-v2
    [*] --> Created: Backtest(data, MyStrategy)
    Created --> Initialized: strategy.init()<br/>Indicators precomputed
    Initialized --> WarmingUp: Skip bars with NaN indicators

    WarmingUp --> Running: First bar where all<br/>indicators have values

    state Running {
        [*] --> BrokerUpdate: broker.next()
        BrokerUpdate --> OrderProcessing: _process_orders()
        OrderProcessing --> EquityLog: Record equity
        EquityLog --> StrategyNext: strategy.next()
        StrategyNext --> [*]: Wait for next bar
    }

    Running --> Finalizing: Last bar reached
    Finalizing --> Completed: Stats computed<br/>Trades finalized

    Running --> Bankrupt: Equity <= 0
    Bankrupt --> Completed: All trades force-closed
```

## Trade Lifecycle

```mermaid
stateDiagram-v2
    [*] --> OrderPlaced: buy() or sell()

    OrderPlaced --> Active: Order filled<br/>Trade opened
    Active --> Active: SL/TP modified<br/>trade.sl = X

    Active --> PartialClose: trade.close(portion=0.5)
    PartialClose --> Active: Reduced trade remains
    PartialClose --> Closed: Portion closed as new trade

    Active --> Closed: trade.close()
    Active --> Closed: position.close()
    Active --> Closed: SL/TP hit
    Active --> Closed: Opposite FIFO close
    Active --> Closed: End of backtest<br/>(finalize_trades=True)

    Closed --> [*]: Moved to closed_trades<br/>P&L finalized with commissions
```

### Trade Properties While Active

| Property | Description |
|----------|-------------|
| `size` | Number of units (negative for short) |
| `entry_price` | Fill price at trade open |
| `exit_price` | `None` while active |
| `pl` | Unrealized P&L: `size * (current_close - entry_price)` |
| `pl_pct` | Unrealized return percentage |
| `value` | `abs(size) * current_price` |
| `sl` | Current stop-loss price (writable) |
| `tp` | Current take-profit price (writable) |
| `tag` | User-defined tracking tag |

### Trade Properties When Closed

| Property | Description |
|----------|-------------|
| `exit_price` | Fill price at trade close |
| `exit_bar` | Bar index when closed |
| `pl` | Realized P&L including commissions |
| `_commissions` | Total commission (entry + exit) |

## Indicator State

Indicators declared via `Strategy.I()` are `_Indicator` objects (subclass of `numpy.ndarray`):

```mermaid
flowchart TD
    INIT["Strategy.init()"] --> CALL["self.I(SMA, self.data.Close, 20)"]
    CALL --> COMPUTE["SMA(data.Close, 20) computed<br/>over full data length"]
    COMPUTE --> WRAP["Wrapped as _Indicator<br/>name, plot, overlay, color, scatter"]
    WRAP --> STORE["Appended to strategy._indicators"]

    NEXT["Strategy.next()"] --> TRUNC["Indicator array truncated<br/>to current bar via _set_length"]
    TRUNC --> ACCESS["self.sma[-1] returns<br/>current bar's value"]
```

The warmup period is determined by finding the first bar where all declared indicators have non-NaN values. The backtest simulation begins at this bar.

---
## See Also
- [README](README.md) — Project overview and quick start
- [Architecture](architecture.md) — System design and components
- [Workflow](workflow.md) — Event flows and processing pipelines
- [Development](development.md) — Development guide and best practices
