# Orbit DevTools: architecture and boundaries

## Responsibility

`orbit-devtools` contains optional local development helpers for Orbit applications. The first
included utility watches one TOML-backed extension-configuration section and submits validated
updates through Core's configuration contract.

```bash
pip install orbit-core orbit-devtools
```

```python
from pathlib import Path

from orbit import Application, ApplicationConfig
from pydantic import BaseModel, ConfigDict

from orbit_devtools import ConfigWatcher


class FeatureSettings(BaseModel):
    model_config = ConfigDict(extra="forbid")
    enabled: bool


app = Application(ApplicationConfig(name="example"))
app.config.register("feature", FeatureSettings(enabled=False))
app.register_config_watcher(
    ConfigWatcher(app.config, "feature", FeatureSettings, Path("settings.toml"))
)
```

The watcher starts and stops with the owning application. It validates the initial file without
changing the already-composed section, and only publishes later updates accepted by Core's
observers. File I/O and parsing run off the event loop; reads are bounded, and errors exposed by
the watcher omit file paths, configuration values, and observer messages. File watching is
opt-in: applications that do not install this package do not poll the filesystem.

## Declared dependencies

The following dependency declarations come from the checked-in manifests. Optional groups and development dependencies are called out separately.

### `pyproject.toml`
- `orbit-core>=0.1.0a1,<0.2`
- Optional `dev` group: `pytest>=8,<10`, `pytest-asyncio>=0.24,<2`, `ruff>=0.8,<1`, `mypy>=1.13,<2`.

Declared dependencies do not mean that optional providers or services are bundled with this package.

## Implementation layout

Representative implementation files in this checkout:

- `src/orbit_devtools/__init__.py`
- `src/orbit_devtools/configuration/__init__.py`
- `src/orbit_devtools/configuration/watcher.py`

## Public contract and scope

## Status

This package is pre-alpha; its API is not stable and it is not yet a published release. It supports
Python 3.11 through 3.14.

## Boundary rules

Keep provider SDKs, credentials, transports, and provider-specific error translation in provider adapters. Keep reusable capability contracts in the matching capability package and lifecycle orchestration in Core. Apply the relevant layer for this repository and preserve the dependency direction shown above.
