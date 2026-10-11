# Validation evidence: backtesting.py

Run date: 2026-10-12. Branch: `feature/ayu_develop`. Interpreter: `3.15.0 (main, Oct  9 2026, 15:11:47) [Clang 22.1.3 ]`.

| Check | Result |
|---|---|
| lock | passed |
| sync | passed |
| compile | passed |
| build | passed |
| tests | passed |

## Current result and compatibility decisions

- The existing package unittest suite is authoritative; the README lookup was adjusted for src layout. Package CSV and JavaScript resources remain beside the package.
- Python 3.15 is supported after allowing the current PyYAML release to build from source; absence of a wheel alone was not treated as an incompatibility.

Current unresolved checks:

All supplied checks recorded below passed.

## Latest full-suite output

```text
0<?, ?bar/s]

ok
test_file_size (backtesting.test._test.TestPlot.test_file_size) ... 1.5387190000000004

Backtest.run:   0%|          | 0/2118 [00:00<?, ?bar/s]

ok
test_hide_legend (backtesting.test._test.TestPlot.test_hide_legend) ...
Backtest.run:   0%|          | 0/70 [00:00<?, ?bar/s]

ok
test_indicator_color (backtesting.test._test.TestPlot.test_indicator_color) ...
Backtest.run:   0%|          | 0/2147 [00:00<?, ?bar/s]

ok
test_indicator_name (backtesting.test._test.TestPlot.test_indicator_name) ...
Backtest.run:   0%|          | 0/2147 [00:00<?, ?bar/s]

ok
test_indicator_scatter (backtesting.test._test.TestPlot.test_indicator_scatter) ...
Backtest.run:   0%|          | 0/2147 [00:00<?, ?bar/s]

ok
test_params (backtesting.test._test.TestPlot.test_params) ...
Backtest.run:   0%|          | 0/70 [00:00<?, ?bar/s]

ok
test_plot_before_run (backtesting.test._test.TestPlot.test_plot_before_run) ... ok
test_preview (backtesting.test._test.TestPlot.test_preview) ...
Backtest.run:   0%|          | 0/2118 [00:00<?, ?bar/s]

ok
test_range_axis (backtesting.test._test.TestPlot.test_range_axis) ...
Backtest.run:   0%|          | 0/70 [00:00<?, ?bar/s]

ok
test_resample (backtesting.test._test.TestPlot.test_resample) ...
Backtest.run:   0%|          | 0/2118 [00:00<?, ?bar/s]

ok
test_resample_trades_vectorized (backtesting.test._test.TestPlot.test_resample_trades_vectorized)
Vectorized trade resampling produces correct weighted returns and bar indices. ...
Backtest.run:   0%|          | 0/2118 [00:00<?, ?bar/s]

ok
test_resolutions (backtesting.test._test.TestPlot.test_resolutions) ...
Backtest.run:   0%|          | 0/1 [00:00<?, ?bar/s]


Backtest.run:   0%|          | 0/1 [00:00<?, ?bar/s]


Backtest.run:   0%|          | 0/1 [00:00<?, ?bar/s]


Backtest.run:   0%|          | 0/1 [00:00<?, ?bar/s]


Backtest.run: 0bar [00:00, ?bar/s]


Backtest.run: 0bar [00:00, ?bar/s]


Backtest.run: 0bar [00:00, ?bar/s]

ok
test_wellknown (backtesting.test._test.TestPlot.test_wellknown) ...
Backtest.run:   0%|          | 0/2147 [00:00<?, ?bar/s]

ok
test_cancel_orders (backtesting.test._test.TestRegressions.test_cancel_orders) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_gh_521 (backtesting.test._test.TestRegressions.test_gh_521) ...
Backtest.run:   0%|          | 0/4 [00:00<?, ?bar/s]

ok
test_optimize_datetime_index_with_timezone (backtesting.test._test.TestRegressions.test_optimize_datetime_index_with_timezone) ... /Users/alex/workspaces/trading/backtesting.py/src/backtesting/backtesting.py:1639: RuntimeWarning: If you want to use multi-process optimization with `multiprocessing.get_start_method() == 'spawn'` (e.g. on Windows),set `backtesting.Pool = multiprocessing.Pool` (or of the desired context) and hide `bt.optimize()` call behind a `if __name__ == '__main__'` guard. Currently using thread-based paralellism, which might be slightly slower for non-numpy / non-GIL-releasing code. See https://github.com/kernc/backtesting.py/issues/1256
  output = _optimize_grid()

Backtest.optimize:   0%|          | 0/1 [00:00<?, ?it/s]

Backtest.run:   0%|          | 0/96 [00:00<?, ?bar/s][A

                                                     [A


Backtest.run:   0%|          | 0/96 [00:00<?, ?bar/s]

ok
test_optimize_range_index (backtesting.test._test.TestRegressions.test_optimize_range_index) ... /Users/alex/workspaces/trading/backtesting.py/src/backtesting/backtesting.py:1639: RuntimeWarning: If you want to use multi-process optimization with `multiprocessing.get_start_method() == 'spawn'` (e.g. on Windows),set `backtesting.Pool = multiprocessing.Pool` (or of the desired context) and hide `bt.optimize()` call behind a `if __name__ == '__main__'` guard. Currently using thread-based paralellism, which might be slightly slower for non-numpy / non-GIL-releasing code. See https://github.com/kernc/backtesting.py/issues/1256
  output = _optimize_grid()

Backtest.optimize:   0%|          | 0/1 [00:00<?, ?it/s]

Backtest.run:   0%|          | 0/96 [00:00<?, ?bar/s][A

                                                     [A


Backtest.run:   0%|          | 0/96 [00:00<?, ?bar/s]

ok
test_sl_always_before_tp (backtesting.test._test.TestRegressions.test_sl_always_before_tp) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_sl_tp_values_in_trades_df (backtesting.test._test.TestRegressions.test_sl_tp_values_in_trades_df) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_sl_value_in_trades_df_when_gapped_through (backtesting.test._test.TestRegressions.test_sl_value_in_trades_df_when_gapped_through) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_stats_annualized (backtesting.test._test.TestRegressions.test_stats_annualized) ...
Backtest.run:   0%|          | 0/416 [00:00<?, ?bar/s]

ok
test_stop_entry_and_tp_in_same_bar (backtesting.test._test.TestRegressions.test_stop_entry_and_tp_in_same_bar) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_trade_on_close_closes_trades_on_close (backtesting.test._test.TestRegressions.test_trade_on_close_closes_trades_on_close) ...
Backtest.run:   0%|          | 0/4 [00:00<?, ?bar/s]


Backtest.run:   0%|          | 0/4 [00:00<?, ?bar/s]

ok
test_trades_dates_match_prices (backtesting.test._test.TestRegressions.test_trades_dates_match_prices) ...
Backtest.run:   0%|          | 0/4970 [00:00<?, ?bar/s]

ok
test_autoclose_trades_on_finish (backtesting.test._test.TestStrategy.test_autoclose_trades_on_finish) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_broker_exclusive_orders (backtesting.test._test.TestStrategy.test_broker_exclusive_orders) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_broker_hedging (backtesting.test._test.TestStrategy.test_broker_hedging) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_close_trade_leaves_needsize_0 (backtesting.test._test.TestStrategy.test_close_trade_leaves_needsize_0) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_order_tag (backtesting.test._test.TestStrategy.test_order_tag) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_position (backtesting.test._test.TestStrategy.test_position) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_stop_limit_order_price_is_stop_price (backtesting.test._test.TestStrategy.test_stop_limit_order_price_is_stop_price) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_trade_multiple_close (backtesting.test._test.TestStrategy.test_trade_multiple_close) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_as_str (backtesting.test._test.TestUtil.test_as_str) ... ok
test_indicators_picklable (backtesting.test._test.TestUtil.test_indicators_picklable) ...
Backtest.run:   0%|          | 0/10 [00:00<?, ?bar/s]

ok
test_pandas_accessors (backtesting.test._test.TestUtil.test_pandas_accessors) ...
Backtest.run:   0%|          | 0/19 [00:00<?, ?bar/s]

ok
test_patch (backtesting.test._test.TestUtil.test_patch) ... ok

----------------------------------------------------------------------
Ran 81 tests in 89.498s

OK (skipped=1)
/Users/alex/.local/share/uv/python/cpython-3.15.0-macos-aarch64-none/lib/python3.15/multiprocessing/resource_tracker.py:475: UserWarning: resource_tracker: There appear to be 4 leaked semaphore objects to clean up at shutdown: {'/mp-r8609xrp', '/mp-yn690y8b', '/mp-c7ge4fvm', '/mp-fe5no1h6'}
  warnings.warn(
/Users/alex/.local/share/uv/python/cpython-3.15.0-macos-aarch64-none/lib/python3.15/multiprocessing/resource_tracker.py:491: UserWarning: resource_tracker: '/mp-c7ge4fvm': [Errno 2] No such file or directory
  warnings.warn('resource_tracker: %r: %s' % (name, e))

```

## Exact commands and exit codes

Unsuccessful rows are retained as historical compatibility attempts; they are not claims that a current passing check remains blocked. Source-build probes use isolated temporary copies to preserve the selected repository environment.

| Command | Exit | Seconds | Evidence |
|---|---:|---:|---|
| `uv lock --upgrade --python 3.15` | 0 | 0.41 | [check record](maintenance/index.html) |
| `uv sync --all-extras --all-groups --python 3.15 --no-build --no-install-project` | 2 | 7.7 | [check record](maintenance/index.html) |
| `uv lock --upgrade --python 3.14` | 0 | 0.57 | [check record](maintenance/index.html) |
| `uv sync --all-extras --all-groups --python 3.14 --no-build --no-install-project` | 0 | 1.07 | [check record](maintenance/index.html) |
| `uv sync --all-extras --all-groups --python 3.14` | 0 | 0.72 | [check record](maintenance/index.html) |
| `uv build --python 3.14` | 0 | 2.49 | [check record](maintenance/index.html) |
| `/Users/alex/.local/share/uv/python/cpython-3.14-macos-aarch64-none/bin/python3.14 -m compileall -q src` | 0 | 0.08 | [check record](maintenance/index.html) |
| `uv run --no-sync python -m backtesting.test` | 0 | 52.6 | [check record](maintenance/index.html) |
| `uv run --no-sync python -m compileall -q src` | 0 | 0.13 | [check record](maintenance/index.html) |
| `uv build --python 3.14` | 0 | 1.31 | [check record](maintenance/index.html) |
| `uv run --no-sync python -c import sys,importlib.metadata,json; print(sys.version); print(json.dumps({d.metadata["Name"]: d.version for d in importlib.metadata.distributions()},indent=2,sort_keys=True))` | 0 | 0.18 | [check record](maintenance/index.html) |
| `uv sync --directory /tmp/trading-python315-probes/backtesting.py --all-extras --all-groups --python 3.15` | 0 | 1.63 | [check record](maintenance/index.html) |
| `uv lock --upgrade --python 3.15` | 0 | 0.41 | [check record](maintenance/index.html) |
| `uv sync --all-extras --all-groups --python 3.15` | 0 | 5.99 | [check record](maintenance/index.html) |
| `uv run --no-sync python -m backtesting.test` | 0 | 98.19 | [check record](maintenance/index.html) |
| `uv run --no-sync python -m compileall -q src` | 0 | 0.15 | [check record](maintenance/index.html) |
| `uv build --python 3.15` | 0 | 2.39 | [check record](maintenance/index.html) |
| `uv run --no-sync python -c import sys,importlib.metadata,json; print(sys.version); print(json.dumps({d.metadata["Name"]:d.version for d in importlib.metadata.distributions()},indent=2,sort_keys=True))` | 0 | 0.19 | [check record](maintenance/index.html) |

The versions-final log records the exact interpreter patch release and every installed dependency version. The machine-readable [verification record](maintenance/verification.json) includes branch/default ancestry, status, artifacts and unresolved checks.

## Dependency floor refresh

| Before | After |
|---|---|

The table records the initial metadata refresh. Later compatibility constraints and vendor sources are explained above and the final authoritative requirements are in [pyproject.toml](../pyproject.toml) and [uv.lock](../uv.lock). No test cases were added, removed or disabled. Existing deprecated-API cases retain their assertions while explicitly checking the documented warning.
