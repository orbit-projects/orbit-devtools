# Orbit DevTools

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

## Status

This package is pre-alpha; its API is not stable and it is not yet a published release. It supports
Python 3.11 through 3.14.

## Documentation

The package-specific guides cover [architecture](docs/architecture/overview.md), [operations and security](docs/operations/README.md), and [development](docs/development/README.md), with [security guidance](docs/security/overview.md). The [documentation index](docs/README.md) links to the full package overview and project policies.

## Development

```bash
python -m pip install -e '.[dev]'
pytest
ruff check .
mypy
```

Licensed under Apache-2.0.

