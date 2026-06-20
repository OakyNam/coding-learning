# 07 - Application Configuration

## Learning Goal

Learn how to load, merge, convert, and validate application configuration in Python so the same code can run safely in development, test, staging, and production.

## Why It Matters

Application configuration is the set of runtime and deployment-specific values your program needs but should not hardcode: host, port, debug mode, database URL, API token, log level, request timeout, feature flags, and similar values.

Good configuration code lets you change deployments without editing source code. Bad configuration code hides defaults, silently accepts broken values, leaks secrets into logs, or makes production depend on a developer's local machine.

## Core Idea

Treat configuration as data that enters the program before the main application starts.

A practical precedence order is:

1. Defaults in code
2. TOML configuration file
3. Environment variables
4. CLI flags

That means each source may override values from the earlier sources. For example, a local `config.toml` might set `port = 9000`, an environment variable might override it with `INVENTORY_PORT=8080`, and a command-line flag might override both with `--port 7000`.

This lesson uses standard-library tools:

- `dataclasses` for a typed `Settings` object
- `tomllib` for reading TOML files in Python 3.11+
- `os.environ` and `os.getenv` for environment variables
- `argparse` for command-line flags
- Explicit conversion functions for booleans, integers, floats, and strings
- Explicit validation that fails fast with a clear error

## Configuration Sources

### Defaults

Defaults should be boring and safe. They are useful for local development and tests, but production-only values should not pretend to have meaningful defaults.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Settings:
    host: str = "127.0.0.1"
    port: int = 8000
    debug: bool = False
    database_url: str = "sqlite:///inventory.db"
    api_token: str = ""
    log_level: str = "INFO"
    timeout_seconds: float = 5.0
```

### TOML File

TOML is a good fit for human-edited configuration because it has real booleans, numbers, strings, arrays, and tables.

Example `config.toml`:

```toml
host = "0.0.0.0"
port = 9000
debug = false
database_url = "postgresql://inventory:localpass@localhost:5432/inventory"
log_level = "DEBUG"
timeout_seconds = 2.5
```

Read it with `tomllib`:

```python
import tomllib
from pathlib import Path


def read_toml(path: Path) -> dict[str, object]:
    with path.open("rb") as file:
        return tomllib.load(file)
```

`tomllib` reads TOML; it does not write TOML. It is available in the standard library starting with Python 3.11.

### Environment Variables

Environment variables are a common deployment boundary. Containers, service managers, CI systems, and cloud platforms can inject them without changing the source tree.

Use a prefix so your app does not accidentally consume unrelated variables:

```powershell
$env:INVENTORY_PORT = "8080"
$env:INVENTORY_LOG_LEVEL = "WARNING"
$env:INVENTORY_API_TOKEN = "dev-token"
```

Important detail: environment variables are strings. Convert them deliberately:

```python
import os

raw_port = os.getenv("INVENTORY_PORT")
port = int(raw_port) if raw_port is not None else 8000
```

`os.environ` behaves like a mapping of the current process environment. `os.getenv("NAME")` is a convenient way to read one value and receive `None` or a chosen default when it is missing.

### CLI Flags

CLI flags are useful for one-off local runs, scripts, and operational overrides:

```powershell
python -m inventory_api --config config.toml --port 7000 --log-level ERROR
```

Use `argparse` to define accepted flags, generate help text, and reject unknown or malformed command-line input.

## Type Conversion And Validation

Configuration values cross a boundary into your program. Do not trust their type or meaning until you check them.

Validate at startup and fail fast when the program cannot run correctly. A web service with an invalid port, impossible timeout, missing production token, or misspelled log level should stop before it starts accepting traffic.

Useful validation rules:

- Boolean strings should be parsed from a small explicit vocabulary such as `true`, `false`, `1`, `0`, `yes`, `no`, `on`, and `off`.
- Ports should be integers from `1` through `65535`.
- Log levels should be one of `DEBUG`, `INFO`, `WARNING`, `ERROR`, or `CRITICAL`.
- Timeouts should be positive numbers.
- Required secrets should be present in environments that need them.
- Error messages should name the broken setting, not expose secret values.

Secrets need special care:

- Do not commit real tokens, passwords, or private URLs with embedded credentials.
- Prefer environment variables or a secrets manager for production secrets.
- Do not print or log secrets during startup.
- When debugging configuration, log whether a secret is configured, not the secret itself.

## Optional INI Comparison

Python also includes `configparser`, which reads INI-style files:

```ini
[inventory]
host = 127.0.0.1
port = 8000
debug = false
```

`configparser` is useful for older projects and simple sectioned files. TOML is often nicer for new Python projects because it has standard types for booleans, integers, floats, arrays, and nested tables. With INI files, many values start as strings and need more manual interpretation.

## Example: `inventory_api` Settings Loader

This example loads settings from defaults, then an optional TOML file, then `INVENTORY_` environment variables, then CLI flags.

```python
from __future__ import annotations

import argparse
import os
import tomllib
from dataclasses import dataclass, replace
from pathlib import Path
from typing import Iterable


class ConfigError(ValueError):
    """Raised when application configuration is missing or invalid."""


@dataclass(frozen=True)
class Settings:
    host: str = "127.0.0.1"
    port: int = 8000
    debug: bool = False
    database_url: str = "sqlite:///inventory.db"
    api_token: str = ""
    log_level: str = "INFO"
    timeout_seconds: float = 5.0


FIELD_TYPES = {
    "host": str,
    "port": int,
    "debug": bool,
    "database_url": str,
    "api_token": str,
    "log_level": str,
    "timeout_seconds": float,
}

ENV_NAMES = {
    "host": "INVENTORY_HOST",
    "port": "INVENTORY_PORT",
    "debug": "INVENTORY_DEBUG",
    "database_url": "INVENTORY_DATABASE_URL",
    "api_token": "INVENTORY_API_TOKEN",
    "log_level": "INVENTORY_LOG_LEVEL",
    "timeout_seconds": "INVENTORY_TIMEOUT_SECONDS",
}

LOG_LEVELS = {"DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"}


def parse_bool(value: object, name: str) -> bool:
    if isinstance(value, bool):
        return value
    if isinstance(value, str):
        normalized = value.strip().lower()
        if normalized in {"1", "true", "yes", "y", "on"}:
            return True
        if normalized in {"0", "false", "no", "n", "off"}:
            return False
    raise ConfigError(f"{name} must be a boolean value")


def convert_value(name: str, value: object) -> object:
    expected_type = FIELD_TYPES[name]
    try:
        if expected_type is bool:
            return parse_bool(value, name)
        if expected_type is int:
            if isinstance(value, bool):
                raise ValueError
            return int(value)
        if expected_type is float:
            if isinstance(value, bool):
                raise ValueError
            return float(value)
        if expected_type is str:
            return str(value)
    except (TypeError, ValueError) as exc:
        raise ConfigError(f"{name} must be {expected_type.__name__}") from exc
    raise ConfigError(f"unsupported setting: {name}")


def apply_overrides(settings: Settings, raw_values: dict[str, object]) -> Settings:
    updates: dict[str, object] = {}
    for name, value in raw_values.items():
        if name not in FIELD_TYPES:
            raise ConfigError(f"unknown setting: {name}")
        if value is not None:
            updates[name] = convert_value(name, value)
    return replace(settings, **updates)


def load_toml_file(path: Path | None) -> dict[str, object]:
    if path is None:
        return {}
    if not path.exists():
        raise ConfigError(f"config file not found: {path}")
    try:
        with path.open("rb") as file:
            data = tomllib.load(file)
    except tomllib.TOMLDecodeError as exc:
        raise ConfigError(f"invalid TOML in {path}: {exc}") from exc
    if not isinstance(data, dict):
        raise ConfigError("config file must contain a TOML table")
    return data


def load_env(prefix_map: dict[str, str] = ENV_NAMES) -> dict[str, object]:
    values: dict[str, object] = {}
    for field_name, env_name in prefix_map.items():
        if env_name in os.environ:
            values[field_name] = os.environ[env_name]
    return values


def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(prog="inventory_api")
    parser.add_argument("--config", type=Path, help="Path to a TOML config file")
    parser.add_argument("--host")
    parser.add_argument("--port")
    parser.add_argument("--debug")
    parser.add_argument("--database-url")
    parser.add_argument("--api-token")
    parser.add_argument("--log-level")
    parser.add_argument("--timeout-seconds")
    return parser


def cli_overrides(args: argparse.Namespace) -> dict[str, object]:
    return {
        "host": args.host,
        "port": args.port,
        "debug": args.debug,
        "database_url": args.database_url,
        "api_token": args.api_token,
        "log_level": args.log_level,
        "timeout_seconds": args.timeout_seconds,
    }


def validate(settings: Settings) -> Settings:
    if not settings.host.strip():
        raise ConfigError("host must not be empty")
    if not 1 <= settings.port <= 65535:
        raise ConfigError("port must be between 1 and 65535")
    if not settings.database_url.strip():
        raise ConfigError("database_url must not be empty")
    if settings.timeout_seconds <= 0:
        raise ConfigError("timeout_seconds must be greater than 0")

    normalized_log_level = settings.log_level.upper()
    if normalized_log_level not in LOG_LEVELS:
        allowed = ", ".join(sorted(LOG_LEVELS))
        raise ConfigError(f"log_level must be one of: {allowed}")

    return replace(settings, log_level=normalized_log_level)


def load_settings(argv: Iterable[str] | None = None) -> Settings:
    parser = build_parser()
    args = parser.parse_args(argv)

    settings = Settings()
    settings = apply_overrides(settings, load_toml_file(args.config))
    settings = apply_overrides(settings, load_env())
    settings = apply_overrides(settings, cli_overrides(args))
    return validate(settings)


def describe_startup(settings: Settings) -> str:
    token_state = "configured" if settings.api_token else "missing"
    return (
        f"inventory_api starting on {settings.host}:{settings.port} "
        f"log_level={settings.log_level} debug={settings.debug} "
        f"api_token={token_state}"
    )


if __name__ == "__main__":
    try:
        loaded_settings = load_settings()
    except ConfigError as exc:
        raise SystemExit(f"configuration error: {exc}") from exc
    print(describe_startup(loaded_settings))
```

The `describe_startup` function deliberately does not print `api_token`. It reports only whether the token exists.

## Example Run

Given this `config.toml`:

```toml
host = "0.0.0.0"
port = 9000
debug = false
database_url = "postgresql://inventory:localpass@localhost:5432/inventory"
log_level = "DEBUG"
timeout_seconds = 2.5
```

And this environment:

```powershell
$env:INVENTORY_PORT = "8080"
$env:INVENTORY_LOG_LEVEL = "WARNING"
$env:INVENTORY_API_TOKEN = "dev-token"
```

Run:

```powershell
python -m inventory_api --config config.toml --port 7000
```

Expected result:

```text
inventory_api starting on 0.0.0.0:7000 log_level=WARNING debug=False api_token=configured
```

Precedence explanation:

- `host` comes from `config.toml` because no environment variable or CLI flag overrides it.
- `port` starts as `8000`, changes to `9000` from TOML, changes to `8080` from `INVENTORY_PORT`, and finally changes to `7000` from `--port`.
- `log_level` starts as `INFO`, changes to `DEBUG` from TOML, then changes to `WARNING` from `INVENTORY_LOG_LEVEL`.
- `api_token` comes from `INVENTORY_API_TOKEN`, and the startup message hides the actual token.
- `debug` comes from TOML and is parsed as a real boolean.

## Common Mistakes

- Treating every setting as a string and hoping the rest of the app will cope.
- Letting invalid configuration fail later in the request path instead of failing at startup.
- Logging the full settings object when it contains secrets.
- Using `bool("false")`, which returns `True` because non-empty strings are truthy.
- Allowing CLI flags, environment variables, and files to override each other in an undocumented order.
- Hardcoding production tokens or database URLs in source code.

## Exercise

Build a configuration loader for `inventory_api`.

Requirements:

1. Create a frozen `Settings` dataclass with `host`, `port`, `debug`, `database_url`, `api_token`, `log_level`, and `timeout_seconds`.
2. Define a `ConfigError` exception.
3. Start with safe defaults.
4. Load optional values from a TOML file passed as `--config`.
5. Load environment variables named with the `INVENTORY_` prefix.
6. Load CLI flags for each setting.
7. Apply precedence in this order: defaults < TOML config file < environment variables < CLI flags.
8. Convert booleans, integers, and floats explicitly.
9. Validate port, log level, timeout, and required non-empty values.
10. Print a startup summary that does not reveal the API token.

Use this sample file:

```toml
host = "0.0.0.0"
port = 9000
debug = true
database_url = "sqlite:///local_inventory.db"
log_level = "DEBUG"
timeout_seconds = 3.0
```

Then test that an environment variable can override the file and a CLI flag can override the environment.

## Worked Answer

One solid answer is the full `inventory_api` loader shown in the example section. When checking your own solution, focus on the behavior:

```powershell
$env:INVENTORY_PORT = "8080"
python -m inventory_api --config config.toml --port 7000
```

The final `port` should be `7000`, because CLI flags have the highest precedence. If you remove `--port 7000`, the final `port` should be `8080` from the environment. If you also remove `INVENTORY_PORT`, the final `port` should be `9000` from the TOML file. If you remove the TOML file value too, the final `port` should be the dataclass default, `8000`.

Your validation should reject examples like these:

```powershell
python -m inventory_api --port 99999
python -m inventory_api --debug maybe
python -m inventory_api --timeout-seconds 0
python -m inventory_api --log-level VERBOSE
```

Each failure should raise or report a clear `ConfigError` that names the invalid setting.

## Next Step

Continue to `08_logging_and_observability.md` and connect this lesson to startup logging: log enough configuration to explain how the app started, but never log secrets.

## Sources Used

- Python documentation: [`tomllib`](https://docs.python.org/3/library/tomllib.html)
- Python documentation: [`os.environ` and `os.getenv`](https://docs.python.org/3/library/os.html)
- Python documentation: [`argparse`](https://docs.python.org/3/library/argparse.html)
- Python documentation: [`dataclasses`](https://docs.python.org/3/library/dataclasses.html)
- Python documentation: [`configparser`](https://docs.python.org/3/library/configparser.html)
- The Twelve-Factor App: [Config](https://12factor.net/config)
- Pydantic documentation: [Settings Management](https://pydantic.dev/docs/validation/latest/concepts/pydantic_settings/)
