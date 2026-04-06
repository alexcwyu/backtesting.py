# Backtesting.py -- Development Guide

## Setup

### Installation

```bash
# From PyPI
pip install backtesting

# Development installation
git clone https://github.com/kernc/backtesting.py
cd backtesting.py
pip install -e '.[doc,test,dev]'
```

### Requirements

- **Python**: 3.9+
- **Core**: numpy >= 1.17.0, pandas >= 0.25.0, bokeh >= 3.0.0
- **Test**: coverage, selenium, pillow
- **Docs**: pdoc3, jupytext, jupyter-nbconvert

## Project Structure

```
backtesting.py/
  src/backtesting/
    __init__.py          # Package exports, Pool configuration
    backtesting.py       # Core: Strategy, Backtest, Order, Trade, Position, _Broker
    lib.py               # Utilities: SignalStrategy, TrailingStrategy, FractionalBacktest,
                         #   MultiBacktest, crossover, barssince, resample_apply, random_ohlc_data
    _stats.py            # Performance statistics computation
    _plotting.py         # Bokeh interactive visualization
    _util.py             # _Data, _Array, _Indicator, SharedMemoryManager
    _version.py          # Version string
    autoscale_cb.js      # JavaScript callback for Bokeh chart autoscaling
    test/
      __init__.py        # Test data (GOOG, EURUSD, BTCUSD) and sample strategies
      _test.py           # Unit test suite
      GOOG.csv           # Google stock OHLCV data
      EURUSD.csv         # EUR/USD forex data
      BTCUSD.csv         # Bitcoin/USD data
  doc/
    examples/            # Jupyter notebook tutorials
    build.sh             # Documentation build script
  pyproject.toml         # Ruff, mypy configuration
  setup.py               # Package metadata and dependencies
```

## Strategy Creation Guide

### Minimal Strategy

```python
from backtesting import Backtest, Strategy

class MyStrategy(Strategy):
    # Class-level parameters (optimizable)
    n_fast = 10
    n_slow = 20

    def init(self):
        """Called once. Precompute indicators here."""
        # self.data has full-length arrays
        close = self.data.Close

        # Declare indicators with self.I()
        self.sma_fast = self.I(SMA, close, self.n_fast)
        self.sma_slow = self.I(SMA, close, self.n_slow)

    def next(self):
        """Called per bar. Make trading decisions here."""
        # Arrays are truncated to current bar
        # Access latest values with [-1]
        if self.sma_fast[-1] > self.sma_slow[-1]:
            if not self.position.is_long:
                self.position.close()  # Close any short
                self.buy()
        elif self.sma_fast[-1] < self.sma_slow[-1]:
            if not self.position.is_short:
                self.position.close()  # Close any long
                self.sell()

# Helper function
def SMA(values, n):
    import pandas as pd
    return pd.Series(values).rolling(n).mean()

# Run
bt = Backtest(data, MyStrategy, cash=10000, commission=0.002)
stats = bt.run()
bt.plot()
```

### Strategy with SL/TP and Position Sizing

```python
class RiskManagedStrategy(Strategy):
    risk_pct = 0.02  # Risk 2% of equity per trade
    atr_mult = 2.0   # SL at 2x ATR

    def init(self):
        close = self.data.Close
        high = self.data.High
        low = self.data.Low

        self.atr = self.I(ATR, high, low, close, 14)
        self.signal = self.I(some_signal_func, close)

    def next(self):
        if self.signal[-1] > 0 and not self.position:
            # Calculate position size based on risk
            atr_val = self.atr[-1]
            sl_distance = atr_val * self.atr_mult
            price = self.data.Close[-1]

            # Size as fraction of equity
            risk_amount = self.equity * self.risk_pct
            units = int(risk_amount / sl_distance)

            if units >= 1:
                self.buy(
                    size=units,
                    sl=price - sl_distance,
                    tp=price + sl_distance * 2  # 2:1 reward/risk
                )
```

### Signal-Based Strategy (Vectorized)

```python
from backtesting.lib import SignalStrategy

class MySignalStrategy(SignalStrategy):
    n = 20

    def init(self):
        super().init()

        close = self.data.Close.s  # pandas Series
        sma = close.rolling(self.n).mean()

        # Entry: +1 for long, -1 for short
        entry = (close > sma).astype(int) - (close < sma).astype(int)
        self.set_signal(entry, plot=True)
```

### Trailing Stop Strategy

```python
from backtesting.lib import TrailingStrategy

class MyTrailingStrategy(TrailingStrategy):
    n_enter = 20
    n_atr = 6

    def init(self):
        super().init()
        self.sma = self.I(SMA, self.data.Close, self.n_enter)
        self.set_trailing_sl(self.n_atr)

    def next(self):
        super().next()  # Updates trailing stops
        if self.data.Close[-1] > self.sma[-1]:
            if not self.position:
                self.buy()
```

## Custom Indicator Development

### Basic Indicator

Indicators are any callable that returns a numpy array (or tuple of arrays) with the same length as the input data:

```python
import pandas as pd
import numpy as np

def RSI(close, period=14):
    """Relative Strength Index"""
    delta = pd.Series(close).diff()
    gain = delta.where(delta > 0, 0).rolling(period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

# Usage in Strategy.init():
self.rsi = self.I(RSI, self.data.Close, 14)
```

### Multi-Output Indicator

```python
def BollingerBands(close, period=20, std=2):
    """Returns (upper, middle, lower) bands"""
    s = pd.Series(close)
    mid = s.rolling(period).mean()
    std_dev = s.rolling(period).std()
    upper = mid + std * std_dev
    lower = mid - std * std_dev
    return upper, mid, lower  # Tuple of arrays

# Usage:
self.bb_upper, self.bb_mid, self.bb_lower = self.I(
    BollingerBands, self.data.Close, 20, 2,
    name=('BB Upper', 'BB Mid', 'BB Lower')
)
```

### Using TA-Lib

```python
import talib

class TALibStrategy(Strategy):
    def init(self):
        close = self.data.Close
        self.rsi = self.I(talib.RSI, close, timeperiod=14)
        self.macd, self.signal, self.hist = self.I(
            talib.MACD, close, name=('MACD', 'Signal', 'Histogram')
        )
```

### Multi-Timeframe Indicators

```python
from backtesting.lib import resample_apply

class MultiTFStrategy(Strategy):
    def init(self):
        # Apply SMA on daily resampled data (from hourly)
        self.daily_sma = resample_apply('D', SMA, self.data.Close, 10)

        # Or manually
        close = self.data.Close.s
        daily = close.resample('D', label='right').agg('last')
        daily_sma = SMA(daily, 10).reindex(close.index).ffill()
        self.daily_sma_manual = self.I(lambda: daily_sma, name='Daily SMA')
```

## Data Source Integration

### From CSV

```python
import pandas as pd

data = pd.read_csv('data.csv', index_col='Date', parse_dates=True)
# Ensure columns: Open, High, Low, Close, Volume
bt = Backtest(data, MyStrategy)
```

### From API (yfinance)

```python
import yfinance as yf

data = yf.download('AAPL', start='2020-01-01', end='2024-01-01')
data.columns = data.columns.droplevel(1)  # Remove multi-level if present
bt = Backtest(data, MyStrategy)
```

### Custom Data with Extra Columns

```python
# Extra columns are accessible via self.data
data['Sentiment'] = sentiment_scores

class SentimentStrategy(Strategy):
    def next(self):
        if self.data.Sentiment[-1] > 0.8:
            self.buy()
```

## Testing

### Running Tests

```bash
# Run all tests
python -m backtesting.test

# With coverage
coverage run -m backtesting.test
coverage combine && coverage report

# Headless (no browser)
BOKEH_BROWSER=none python -m backtesting.test
```

### Linting

```bash
# Ruff (configured in pyproject.toml, 100-char line length)
ruff check backtesting

# Type checking
mypy --no-warn-unused-ignores backtesting
```

### Writing Tests

Tests are in `src/backtesting/test/_test.py` using `unittest`. Sample data is available:

```python
from backtesting.test import GOOG, EURUSD, BTCUSD, SmaCross, SMA

# GOOG, EURUSD, BTCUSD are pd.DataFrames with OHLCV data
# SmaCross is a sample strategy class
# SMA is a simple moving average function
```

## Common Patterns

### Parameter Optimization

```python
stats = bt.optimize(
    n1=range(5, 30, 5),
    n2=range(10, 70, 5),
    maximize='Sharpe Ratio',
    constraint=lambda p: p.n1 < p.n2
)
print(stats._strategy)  # Shows optimal parameters
```

### Analyzing Trade Subsets

```python
from backtesting.lib import compute_stats

stats = bt.run()
long_trades = stats._trades[stats._trades.Size > 0]
long_stats = compute_stats(
    stats=stats,
    trades=long_trades,
    data=data,
    risk_free_rate=0.02
)
```

### Fractional / Crypto Trading

```python
from backtesting.lib import FractionalBacktest

bt = FractionalBacktest(btc_data, MyStrategy, fractional_unit=1/1e6)
stats = bt.run()
```

### Multi-Asset Comparison

```python
from backtesting.lib import MultiBacktest
from backtesting.test import EURUSD, BTCUSD, SmaCross

btm = MultiBacktest([EURUSD, BTCUSD], SmaCross)
stats_df = btm.run(fast=10, slow=20)
```

## Configuration Reference

### Backtest Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `data` | `pd.DataFrame` | *(required)* | OHLC(V) DataFrame with DatetimeIndex or RangeIndex. Required columns: Open, High, Low, Close. |
| `strategy` | `Type[Strategy]` | *(required)* | Strategy subclass (not an instance). |
| `cash` | `float` | `10_000` | Initial cash balance. |
| `spread` | `float` | `0.0` | Constant bid-ask spread as a fraction of price (e.g., `0.0002` for 0.2 permille). Applied once per trade. |
| `commission` | `float \| tuple \| callable` | `0.0` | Commission rate. A float is treated as a fraction of order value (e.g., `0.01` = 1%). A tuple `(fixed, relative)` combines a fixed fee with a percentage. A callable `func(order_size, price) -> float` allows custom models. Applied at both entry and exit. |
| `margin` | `float` | `1.0` | Required margin ratio for leveraged accounts. Set to `1/leverage` (e.g., `0.02` for 50:1 leverage). |
| `trade_on_close` | `bool` | `False` | If `True`, market orders fill at the current bar's close instead of the next bar's open. |
| `hedging` | `bool` | `False` | If `True`, allows simultaneous long and short positions. If `False`, opposite orders close existing trades FIFO. |
| `exclusive_orders` | `bool` | `False` | If `True`, each new order auto-closes the previous trade, keeping at most one active position. |
| `finalize_trades` | `bool` | `False` | If `True`, still-open trades are closed on the last bar and included in statistics. |

### Strategy.buy() / Strategy.sell() Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `size` | `float` | `None` | Order size. Values 0-1 are treated as a fraction of equity; values >= 1 as absolute units. `None` uses all available equity. |
| `limit` | `float` | `None` | Limit price. Order fills when price reaches this level. |
| `stop` | `float` | `None` | Stop price. Order activates when price reaches this level. |
| `sl` | `float` | `None` | Stop-loss price. Automatically closes the trade if price hits this level. |
| `tp` | `float` | `None` | Take-profit price. Automatically closes the trade if price hits this level. |

### Backtest.optimize() Key Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `maximize` | `str \| callable` | `'SQN'` | Statistic to maximize (e.g., `'Sharpe Ratio'`, `'Equity Final [$]'`), or a callable that receives a stats Series. |
| `constraint` | `callable` | `None` | Function receiving a parameter namespace, returning `True` to keep the combination. |
| `return_heatmap` | `bool` | `False` | If `True`, returns a Series indexed by parameter tuples alongside the best stats. |
| `return_optimization` | `bool` | `False` | If `True`, returns the full optimization results DataFrame. |
| `random_state` | `int` | `None` | Seed for reproducible random search. |
| `max_tries` | `int \| float` | `None` | Maximum parameter combinations to try. A float 0-1 is treated as a fraction of the total space. |

## Troubleshooting

### 1. `TypeError: 'strategy' must be a Strategy sub-type`
**Cause**: Passing a Strategy instance instead of the class itself.
**Fix**: Pass the class, not an instance: `Backtest(data, MyStrategy)` -- not `Backtest(data, MyStrategy())`.

### 2. `ValueError` or empty results with no trades
**Cause**: Indicators have a warmup period that consumes initial bars; if data is too short, `next()` is never called with valid signals.
**Fix**: Ensure the dataset has significantly more rows than the longest indicator period. Check that your `next()` logic actually triggers buy/sell under the data conditions.

### 3. `KeyError: 'Open'` (or High, Low, Close)
**Cause**: DataFrame columns are lowercase or differently named.
**Fix**: Rename columns to match the expected capitalization: `df.columns = ['Open', 'High', 'Low', 'Close', 'Volume']`.

### 4. Orders not filling / unexpected fill prices
**Cause**: By default, market orders fill at the next bar's open price, not the current bar's close.
**Fix**: Set `trade_on_close=True` in the Backtest constructor if you want fills at the current bar's close. For limit/stop orders, ensure the target price is actually reached by the bar's High/Low range.

### 5. `self.sell()` does not close a long position
**Cause**: Without `exclusive_orders=True`, `sell()` opens a new short position rather than closing the existing long.
**Fix**: Use `self.position.close()` to explicitly close the current position, or set `exclusive_orders=True` in the Backtest constructor.

### 6. Optimization hangs or crashes on Windows
**Cause**: Python's `spawn` multiprocessing method on Windows requires the `if __name__ == '__main__'` guard.
**Fix**: Wrap your `optimize()` call inside `if __name__ == '__main__':`. Alternatively, set `backtesting.Pool = multiprocessing.Pool` before calling optimize.

### 7. Bokeh plot not rendering in Jupyter
**Cause**: Bokeh output mode not configured for notebooks.
**Fix**: Call `from bokeh.io import output_notebook; output_notebook()` before `bt.plot()`. For headless environments, set `BOKEH_BROWSER=none`.

### 8. Fractional position sizes are rounded to zero
**Cause**: The default broker rounds sizes to integers. With high-priced assets and low capital, this can round to zero.
**Fix**: Use `FractionalBacktest` from `backtesting.lib` for crypto or fractional trading: `from backtesting.lib import FractionalBacktest`.

## Security Considerations

### API Keys and Credentials
- Never hardcode API keys, broker credentials, or exchange secrets in strategy files. Use environment variables or a secrets manager.
- The framework itself does not handle external API connections, but data loading scripts often do. Keep credentials out of version control.

### Data Integrity
- Validate input data for NaN values, gaps, and outliers before running backtests. Corrupt data can produce misleading statistics.
- Use `data.dropna()` or forward-fill judiciously. Be aware that filling gaps can introduce look-ahead bias.

### Look-Ahead Bias
- Indicators declared in `init()` compute over the full dataset but are progressively revealed in `next()`. Custom indicator functions that inadvertently access future data will produce unrealistically good results.
- When using `resample_apply()`, ensure the resampled values are properly lagged to avoid peeking at intra-period data.

### Survivorship Bias
- Backtesting only on currently listed securities ignores delisted or bankrupt companies. Source data that includes delistings for realistic results.

### Multiprocessing and Shared State
- The optimization engine uses multiprocessing. Avoid global mutable state in strategies, as child processes get copies (not references) of global objects.
- On systems with `fork` semantics, file descriptors and locks may be inherited unexpectedly. Prefer `spawn` or ensure cleanup in `init()`.

### Pickle and Serialization
- Optimization results and strategy objects may be pickled when using multiprocessing. Do not store sensitive data (credentials, PII) as strategy attributes, as they could be serialized to disk in temporary files.

---
## See Also
- [README](README.md) — Project overview and quick start
- [Architecture](architecture.md) — System design and components
- [Workflow](workflow.md) — Event flows and processing pipelines
- [State Management](state-management.md) — State lifecycle and data models
