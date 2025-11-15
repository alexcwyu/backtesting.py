# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**Backtesting.py** is a Python framework for backtesting trading strategies with candlestick data. It provides a simple, well-documented API for defining strategies, running backtests, optimizing parameters, and visualizing results with interactive Bokeh charts.

**Documentation**: https://kernc.github.io/backtesting.py/doc/backtesting/
**Repository**: https://github.com/kernc/backtesting.py

## Installation & Environment Setup

### Development Installation

```bash
# Clone and install with all development dependencies
git clone https://github.com/kernc/backtesting.py
cd backtesting.py
pip install -e '.[doc,test,dev]'
```

### Requirements

- **Python**: 3.9+ (enforced in setup.py:4)
- **Core Dependencies**: numpy >= 1.17.0, pandas >= 0.25.0, bokeh >= 3.0.0
- **Optional Dependencies**: See setup.py:38-58 for doc, test, and dev extras

## Common Development Commands

### Testing

```bash
# Run all tests (uses unittest discovery)
python -m backtesting.test

# Run tests with coverage
coverage run -m backtesting.test
coverage combine && coverage report

# Test with environment variable (for headless environments)
BOKEH_BROWSER=none python -m backtesting.test
```

The test suite is located in `backtesting/test/_test.py` with test data in CSV files (GOOG.csv, EURUSD.csv, BTCUSD.csv).

### Linting & Type Checking

```bash
# Code style checking (uses flake8)
flake8 backtesting setup.py

# Type checking (uses mypy)
mypy --no-warn-unused-ignores backtesting

# Modern linting with ruff (configured in pyproject.toml)
# Ruff configuration: 100 char line length, excludes doc/examples
```

Ruff configuration in pyproject.toml:1-38 ignores certain rules (UP006, UP007, UP009, N802, N806, C901, B008, B011, RUF002) and enforces I, E, F, W, UP, N, C, B, T, RUF, YTT checks.

### Documentation

```bash
# Build full documentation (API reference + example notebooks)
doc/build.sh

# This will:
# 1. Generate API docs with pdoc3
# 2. Convert .py examples to Jupyter notebooks
# 3. Execute notebooks and convert to HTML
# 4. Validate all links

# Requirements: pdoc3, jupytext, jupyter-nbconvert
pip install -e '.[doc]'
```

Documentation is built to `doc/build/` directory. API reference is generated from docstrings in code. Examples are Jupyter notebooks in `doc/examples/` that are maintained as `.py` files (using jupytext).

## Code Architecture

### Core Components

The framework consists of four main modules in `backtesting/`:

1. **backtesting.py** (1500+ lines): Core engine with main classes
   - `Strategy` (base class): Define `init()` for indicator setup, `next()` for per-bar logic
   - `Backtest`: Main entry point for running backtests and optimizations
   - `Order`: Represents placed orders (market, limit, stop, SL/TP)
   - `Trade`: Represents filled orders with P&L tracking
   - `Position`: Current position state (size, P&L, closing logic)
   - `_Broker`: Internal broker simulation engine

2. **lib.py**: Utility functions and composable strategy base classes
   - Helper functions: `crossover()`, `barssince()`, `plot_heatmaps()`, `random_ohlc_data()`
   - Strategy mixins: `SignalStrategy`, `TrailingStrategy`
   - Multi-asset support: `MultiBacktest`, `FractionalBacktest`
   - Aggregation rules: `OHLCV_AGG`, `TRADES_AGG` for resampling data

3. **_stats.py**: Statistics computation and performance metrics
   - `compute_stats()`: Generates comprehensive performance report
   - Metrics: Sharpe ratio, Sortino ratio, Calmar ratio, drawdowns, win rate, etc.
   - `compute_drawdown_duration_peaks()`: Drawdown analysis

4. **_plotting.py**: Interactive Bokeh visualization
   - `plot()`: Generates interactive charts with OHLC, indicators, trades, P&L
   - Autoscaling, zooming, panning, legend toggling
   - JavaScript callback in `autoscale_cb.js`

### Strategy Development Pattern

Strategies inherit from `Strategy` and implement two methods:

```python
class MyStrategy(Strategy):
    def init(self):
        # Called once at start - precompute indicators
        # Access full data arrays: self.data.Close, self.data.Open, etc.
        # Use self.I() to declare indicators
        self.sma = self.I(SMA, self.data.Close, 20)

    def next(self):
        # Called for each bar - make trading decisions
        # Access current values: self.data.Close[-1], self.sma[-1]
        # Place orders: self.buy(), self.sell()
        # Manage positions: self.position, self.orders, self.trades
```

Key concepts:
- **Indicator Declaration**: Use `self.I(func, *args, **kwargs)` to wrap indicator functions. The framework handles progressive revelation of values in `next()`.
- **Data Access**: In `init()`, arrays are full-length. In `next()`, arrays reveal values progressively (only up to current bar).
- **Order Placement**: `buy()`/`sell()` return `Order` objects. Orders are filled when conditions are met.
- **Position Management**: Access `self.position` to query current position, `self.trades` for all trades.

### Data Structure

Input data is a pandas DataFrame with OHLC(V) columns and datetime index:
- Required columns: Open, High, Low, Close
- Optional: Volume (or any custom columns)
- Index: DatetimeIndex

Internally converted to `_Data` structure (see backtesting.py:276) which provides numpy array access for performance while maintaining index information.

### Broker Simulation

The `_Broker` class (backtesting.py:743) simulates order execution:
- **Event-driven**: Processes orders bar-by-bar
- **Order types**: Market, limit, stop, stop-limit, SL/TP (Good 'Til Canceled)
- **Execution timing**:
  - Default: Market orders fill at next bar's open
  - `trade_on_close=True`: Fill at current bar's close
- **Position sizing**: Supports fractional (0-1 = % of equity) or absolute (>= 1 = number of units)
- **Margin trading**: Configurable with `margin` parameter
- **Commission/spread**: Applied to each trade

### Optimization Engine

The `Backtest.optimize()` method (backtesting.py:1300+) supports parameter optimization:
- **Grid search**: Tests all combinations of parameters
- **Constraint functions**: Filter parameter combinations
- **Parallel execution**: Uses `backtesting.Pool` (configurable, defaults to multiprocessing)
- **Visualization**: `plot_heatmaps()` for 2D/3D parameter space visualization
- **Maximization target**: Any stats key (e.g., 'Equity Final [$]', 'Sharpe Ratio')

Note: On Windows with spawn multiprocessing, set `backtesting.Pool = multiprocessing.Pool` and guard `optimize()` with `if __name__ == '__main__'` (see backtesting/__init__.py:74).

## Project Conventions

### Code Style

- **Line length**: 100 characters (pyproject.toml:19)
- **Naming**:
  - Classes: PascalCase
  - Functions: snake_case
  - Private: Leading underscore (_private_method)
  - Special exceptions: `Strategy.I()` method (uppercase), variable names `l`, `h` (allowed in ruff config)
- **Type hints**: Used throughout, validated with mypy
- **Docstrings**: Markdown format, compatible with pdoc3

### Testing Requirements

- Unit tests for new/changed functionality in `backtesting/test/_test.py`
- Test data available as module attributes: `backtesting.test.GOOG`, `EURUSD`, `BTCUSD`
- Tests must pass before PR: `python -m backtesting.test`
- Aim for high coverage (tracked with codecov)

### Commit Messages

Follow explicit commit message style (see CONTRIBUTING.md:82):
- Reference NumPy's development workflow for inspiration
- Clear, descriptive messages
- Link to issues when applicable

## Common Pitfalls

### 1. Strategy Indicator Access
In `next()`, indicators reveal values progressively. Always access most recent value with `[-1]`:
```python
# Correct
if self.sma[-1] > self.data.Close[-1]:
    self.buy()

# Wrong - accesses all values, not just current
if self.sma > self.data.Close:
    self.buy()
```

### 2. Position vs Trade Closing
- `self.sell()` does NOT close `self.buy()` trades unless `exclusive_orders=True`
- Use `self.position.close()` to close current position
- Use `trade.close()` to close specific trade

### 3. Windows Multiprocessing
On Windows, `optimize()` requires `if __name__ == '__main__'` guard due to spawn multiprocessing method (backtesting/__init__.py:78).

### 4. Data Requirements
- DataFrame must have DatetimeIndex (not integer index)
- Column names are case-sensitive (Open, High, Low, Close)
- Data must be sorted by time (ascending)

## Example Workflows

### Basic Backtest
```python
from backtesting import Backtest, Strategy
from backtesting.lib import crossover
from backtesting.test import SMA, GOOG

class SmaCross(Strategy):
    n1 = 10
    n2 = 20

    def init(self):
        self.sma1 = self.I(SMA, self.data.Close, self.n1)
        self.sma2 = self.I(SMA, self.data.Close, self.n2)

    def next(self):
        if crossover(self.sma1, self.sma2):
            self.buy()
        elif crossover(self.sma2, self.sma1):
            self.position.close()

bt = Backtest(GOOG, SmaCross, commission=.002, exclusive_orders=True)
stats = bt.run()
bt.plot()
```

### Parameter Optimization
```python
stats = bt.optimize(
    n1=range(5, 30, 5),
    n2=range(10, 70, 5),
    maximize='Sharpe Ratio',
    constraint=lambda p: p.n1 < p.n2
)
```

### Custom Indicators
```python
def init(self):
    # Use any indicator library (TA-Lib, pandas-ta, etc.)
    close = pd.Series(self.data.Close)
    self.rsi = self.I(lambda: ta.rsi(close, length=14).values)
```

## Support & Resources

- **Documentation**: https://kernc.github.io/backtesting.py/doc/backtesting/
- **Tutorials**: doc/examples/ (Quick Start, Multiple Time Frames, ML Trading, Optimization)
- **Issues**: https://github.com/kernc/backtesting.py/issues
- **Discussions**: https://github.com/kernc/backtesting.py/discussions
- **Change Log**: CHANGELOG.md (for recent changes and breaking changes)
- **FAQ**: Search issues with `label:question` or discussions

When reporting bugs, follow CONTRIBUTING.md guidelines: provide minimal working example, full traceback, and use fenced code blocks.
