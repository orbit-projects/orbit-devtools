# Orbit DevTools: operations and security

This guide organizes runtime behavior documented by the package. It does not certify production readiness. Verify provider/client versions, permissions, transport security, limits, and failure behavior in the target environment before release.

## Configuration surface

Environment names found in the package README:

The package README does not name `ORBIT_*` variables. Use its typed constructors and application configuration, and confirm exact runtime inputs in the implementation before deployment.

Use the package README's constructor and deployment examples. Store credentials in a secret manager and avoid logging credentials, raw provider errors, request data, or opaque cursors.

## Lifecycle, failure behavior, and limits

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

## Production validation

Validate startup/shutdown cleanup, timeout and cancellation behavior, concurrency and payload bounds where applicable, secret rotation and least-privilege access, data durability, backup/restore, and failover against the selected provider. Do not infer distributed or durable guarantees from an in-process API or fake-client tests.
