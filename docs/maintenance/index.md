# backtesting.py: portable verification records

Full raw logs are retained in the workspace maintenance evidence, not distributed here. Hashes identify those original logs. A successful build or selected test subset does not imply a full test-suite pass. Commands containing <workspace> describe the original execution environment.

[Machine-readable records](verification.json) · [Repository validation summary](../validation.md)

## Executed commands

| Check | Exit | Duration | Command | Outcomes |
| --- | ---: | ---: | --- | --- |
| build | 0 | 2.49s | `uv build --python 3.14` | Successfully built dist/backtesting-0.3.3.tar.gz<br>Successfully built dist/backtesting-0.3.3-py3-none-any.whl |
| build-final | 0 | 1.31s | `uv build --python 3.14` | Successfully built dist/backtesting-0.3.3.tar.gz<br>Successfully built dist/backtesting-0.3.3-py3-none-any.whl |
| build-selected-python | 0 | 2.39s | `uv build --python 3.15` | Successfully built dist/backtesting-0.3.3.tar.gz<br>Successfully built dist/backtesting-0.3.3-py3-none-any.whl |
| compile | 0 | 0.08s | `/Users/alex/.local/share/uv/python/cpython-3.14-macos-aarch64-none/bin/python3.14 -m compileall -q src` | See exit status; no test-count assertion. |
| compile-final | 0 | 0.13s | `uv run --no-sync python -m compileall -q src` | See exit status; no test-count assertion. |
| compile-selected-python | 0 | 0.15s | `uv run --no-sync python -m compileall -q src` | See exit status; no test-count assertion. |
| lock-3.14 | 0 | 0.57s | `uv lock --upgrade --python 3.14` | See exit status; no test-count assertion. |
| lock-3.15 | 0 | 0.41s | `uv lock --upgrade --python 3.15` | See exit status; no test-count assertion. |
| lock-python315-source-supported | 0 | 0.41s | `uv lock --upgrade --python 3.15` | See exit status; no test-count assertion. |
| python315-source-probe | 0 | 1.63s | `uv sync --directory /tmp/trading-python315-probes/backtesting.py --all-extras --all-groups --python 3.15` | See exit status; no test-count assertion. |
| sync-3.14 | 0 | 0.72s | `uv sync --all-extras --all-groups --python 3.14` | See exit status; no test-count assertion. |
| sync-python315-source-supported | 0 | 5.99s | `uv sync --all-extras --all-groups --python 3.15` | See exit status; no test-count assertion. |
| tests | 0 | 52.6s | `uv run --no-sync python -m backtesting.test` | See exit status; no test-count assertion. |
| tests-python315-source-supported | 0 | 98.19s | `uv run --no-sync python -m backtesting.test` | See exit status; no test-count assertion. |
| versions-final | 0 | 0.19s | `uv run --no-sync python -c import sys,importlib.metadata,json; print(sys.version); print(json.dumps({d.metadata["Name"]:d.version for d in importlib.metadata.distributions()},indent=2,sort_keys=True))` | See exit status; no test-count assertion. |
| wheel-probe-3.14 | 0 | 1.07s | `uv sync --all-extras --all-groups --python 3.14 --no-build --no-install-project` | See exit status; no test-count assertion. |
| wheel-probe-3.15 | 2 | 7.7s | `uv sync --all-extras --all-groups --python 3.15 --no-build --no-install-project` | See exit status; no test-count assertion. |
