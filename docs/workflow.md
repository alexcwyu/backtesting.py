# Backtesting.py -- Workflow

## Backtesting Pipeline

```mermaid
sequenceDiagram
    participant U as User
    participant BT as Backtest
    participant S as Strategy
    participant BR as _Broker
    participant D as _Data
    participant ST as Stats

    U->>BT: bt = Backtest(data, MyStrategy, cash=10000)
    U->>BT: stats = bt.run()

    Note over BT: 1. Initialization
    BT->>D: Create _Data(dataframe)
    BT->>BR: Create _Broker(data, cash, spread, commission, margin)
    BT->>S: Create MyStrategy(broker, data, params)

    Note over BT: 2. Indicator Precomputation
    BT->>S: strategy.init()
    S->>S: self.sma = self.I(SMA, self.data.Close, 20)
    Note over S: Full-length arrays available

    Note over BT: 3. Determine warmup period
    BT->>BT: Skip bars where indicators have NaN

    Note over BT: 4. Bar-by-bar simulation
    loop For bar i = warmup to len(data)
        BT->>D: data._set_length(i + 1)
        BT->>BR: broker.next()
        BR->>BR: _process_orders()
        BR->>BR: equity[i] = cash + unrealized_pl
        BT->>S: strategy.next()
        Note over S: Arrays truncated to bar i
        S->>BR: self.buy() / self.sell()
    end

    Note over BT: 5. Finalization
    BT->>BT: Optionally close active trades
    BT->>ST: compute_stats(trades, equity, data)
    ST-->>U: pd.Series with 30+ metrics
```

## Strategy Execution Flow

```mermaid
flowchart TD
    A[Strategy.init called] --> B{Declare indicators}
    B --> C[self.I func, args]
    C --> D[Indicator array computed<br/>over full data length]
    D --> E[Wrapped in _Indicator<br/>with plot metadata]
    E --> F{More indicators?}
    F -->|Yes| C
    F -->|No| G[Warmup period calculated<br/>from indicator NaN lengths]

    G --> H[Strategy.next loop begins]
    H --> I[Data arrays truncated<br/>to current bar]
    I --> J{Evaluate trading logic}
    J -->|Buy signal| K[self.buy size, limit, stop, sl, tp]
    J -->|Sell signal| L[self.sell size, limit, stop, sl, tp]
    J -->|Close position| M[self.position.close]
    J -->|Modify SL/TP| N[trade.sl = price<br/>trade.tp = price]
    J -->|No action| O[Continue]

    K --> P[Order added to queue]
    L --> P
    M --> P
    N --> P
    O --> H
    P --> H
```

## Order Processing Flow

```mermaid
flowchart TD
    START[_process_orders called<br/>each bar] --> ITER[Iterate order queue]

    ITER --> CHK_STOP{Has stop price?}
    CHK_STOP -->|Yes| STOP_HIT{Stop hit?<br/>High >= stop long<br/>Low <= stop short}
    STOP_HIT -->|No| SKIP[Skip order this bar]
    STOP_HIT -->|Yes| CONVERT[Convert to market/limit<br/>Remove stop price]
    CHK_STOP -->|No| CHK_LIMIT

    CONVERT --> CHK_LIMIT{Has limit price?}
    CHK_LIMIT -->|Yes| LIMIT_HIT{Limit hit?<br/>Low <= limit long<br/>High >= limit short}
    LIMIT_HIT -->|No| SKIP
    LIMIT_HIT -->|Yes| CALC_PRICE[Price = min/max<br/>of stop and limit]
    CHK_LIMIT -->|No| MARKET[Price = prev close<br/>or current open]

    CALC_PRICE --> SIZE
    MARKET --> SIZE

    SIZE{Fractional size?<br/>0 < size < 1}
    SIZE -->|Yes| CALC_SIZE[size = margin_available<br/> x leverage x fraction<br/> / adjusted_price]
    SIZE -->|No| USE_SIZE[Use absolute size]

    CALC_SIZE --> PARENT
    USE_SIZE --> PARENT

    PARENT{Is SL/TP order?}
    PARENT -->|Yes| REDUCE[Reduce/close parent trade]
    PARENT -->|No| HEDGE{Hedging enabled?}

    HEDGE -->|No| FIFO[Close opposite trades FIFO]
    HEDGE -->|Yes| MARGIN_CHK

    FIFO --> MARGIN_CHK{Sufficient margin?}
    MARGIN_CHK -->|No| CANCEL[Cancel order<br/>Warning issued]
    MARGIN_CHK -->|Yes| OPEN[Open new trade]

    OPEN --> SLTP{SL/TP specified?}
    SLTP -->|Yes| BRACKET[Create contingent<br/>SL/TP orders]
    SLTP -->|No| DONE[Order processed]
    BRACKET --> DONE
    REDUCE --> DONE

    SKIP --> NEXT[Next order]
    CANCEL --> NEXT
    DONE --> NEXT
    NEXT --> ITER
```

## Data Feed Handling

Backtesting.py uses a simple, pandas-based data model:

```mermaid
flowchart LR
    subgraph Input Data
        RAW[pandas DataFrame<br/>DatetimeIndex<br/>Open, High, Low, Close, Volume<br/>+ optional custom columns]
    end

    subgraph Validation
        RAW --> V1{Has OHLCV columns?}
        V1 -->|No| ERR1[ValueError]
        V1 -->|Yes| V2{NaN in OHLC?}
        V2 -->|Yes| ERR2[ValueError]
        V2 -->|No| V3{Monotonic index?}
        V3 -->|No| SORT[Sort ascending + warning]
        V3 -->|Yes| V4{DatetimeIndex?}
        V4 -->|No| WARN[Warning: simple periods assumed]
    end

    subgraph Internal
        SORT --> WRAP
        WARN --> WRAP
        V4 -->|Yes| WRAP[_Data wrapper created]
        WRAP --> ARRAYS[numpy arrays per column]
        ARRAYS --> CACHE[Cached sliced views<br/>via _set_length]
    end
```

Data is accessed in `Strategy` via `self.data`:
- `self.data.Close` -- numpy array (truncated in `next()`)
- `self.data.Close.s` -- pandas Series with datetime index
- `self.data.df` -- full DataFrame slice

## Optimization Flow

```mermaid
flowchart TD
    A[bt.optimize<br/>param1=range..., param2=range...<br/>maximize='Sharpe Ratio'] --> B[Generate parameter grid<br/>cartesian product]
    B --> C{Constraint function?}
    C -->|Yes| D[Filter valid combinations]
    C -->|No| E[All combinations]
    D --> F
    E --> F[Shared memory:<br/>Copy data to SHM]

    F --> G[Multiprocessing Pool]
    G --> H1[Worker 1:<br/>Run backtest with params_1]
    G --> H2[Worker 2:<br/>Run backtest with params_2]
    G --> HN[Worker N:<br/>Run backtest with params_N]

    H1 --> I[Collect results]
    H2 --> I
    HN --> I

    I --> J{return_heatmap?}
    J -->|Yes| K[Build MultiIndex Series<br/>of maximize values]
    J -->|No| L[Return best stats]

    K --> M[plot_heatmaps<br/>Bokeh grid of 2D heatmaps]

    L --> N[Best strategy parameters<br/>+ full stats]
```

### Optimization Example

```python
stats, heatmap = bt.optimize(
    n1=range(5, 30, 5),
    n2=range(10, 70, 5),
    maximize='Sharpe Ratio',
    constraint=lambda p: p.n1 < p.n2,
    return_heatmap=True
)
```

The optimizer:
1. Generates all `(n1, n2)` combinations where `n1 < n2`
2. Runs backtests in parallel across CPU cores
3. Returns the parameter set that maximizes the Sharpe Ratio
4. Optionally returns a heatmap Series for visualization

## Walk-Forward Analysis

Backtesting.py does not include built-in walk-forward analysis, but it can be implemented manually:

```python
from backtesting import Backtest, Strategy

# Split data into windows
for train_start, train_end, test_start, test_end in windows:
    train_data = data[train_start:train_end]
    test_data = data[test_start:test_end]

    # Optimize on training window
    bt_train = Backtest(train_data, MyStrategy)
    stats = bt_train.optimize(n1=range(5, 50), n2=range(10, 100),
                               maximize='Sharpe Ratio',
                               constraint=lambda p: p.n1 < p.n2)

    # Test on out-of-sample window
    bt_test = Backtest(test_data, MyStrategy)
    oos_stats = bt_test.run(n1=stats._strategy.n1, n2=stats._strategy.n2)
```

## Monte Carlo Simulation

The `random_ohlc_data()` generator enables robustness testing:

```python
from backtesting.lib import random_ohlc_data

generator = random_ohlc_data(real_data, frac=1.0, random_state=42)
results = []
for _ in range(1000):
    random_data = next(generator)
    bt = Backtest(random_data, MyStrategy)
    stats = bt.run()
    results.append(stats['Sharpe Ratio'])
```

---
## See Also
- [README](README.md) — Project overview and quick start
- [Architecture](architecture.md) — System design and components
- [State Management](state-management.md) — State lifecycle and data models
- [Development](development.md) — Development guide and best practices
