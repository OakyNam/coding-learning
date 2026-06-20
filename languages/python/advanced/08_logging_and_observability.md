# 08 - Logging and Observability

## Learning Goal

Learn how Python applications report what they are doing, why logging is different from print debugging, and how logs connect to broader observability signals like metrics and traces.

By the end of this lesson, you should be able to:

- Configure logging for an application.
- Choose useful log levels and event names.
- Attach contextual data such as request IDs.
- Avoid common production logging mistakes.
- Explain how logs, metrics, traces, spans, and correlation IDs work together.

## Why Logging Is Not Print Debugging

`print()` is useful while learning or running a tiny script by hand. It sends text to standard output right now, with no severity, no routing, no structured context, and no easy way to turn one part of the program up or down.

Logging is application telemetry. A log event says, "something meaningful happened," and carries metadata that helps another person or system understand it later. Good logs can be filtered by level, routed to different destinations, formatted consistently, enriched with request data, and collected by observability tools.

Use `print()` for quick experiments. Use logging for programs you may need to debug, operate, audit, or support.

## Python Logging Architecture

Python's `logging` package is built from a few cooperating objects:

- `Logger`: the object your code calls, usually created with `logging.getLogger(__name__)`.
- `Handler`: sends log records somewhere, such as the console, a file, a socket, or an external collector.
- `Formatter`: turns a `LogRecord` into text.
- `Filter`: accepts, rejects, or enriches records before they are emitted.
- `LogRecord`: the event object created for each logging call. It contains the message, level, logger name, timestamp, pathname, line number, exception info, and any extra fields.

Loggers are arranged in a dotted-name hierarchy. A logger named `shop.orders.payment` is a child of `shop.orders`, which is a child of `shop`. The root logger sits at the top.

Important behaviors:

- A named logger should usually match the module name: `logger = logging.getLogger(__name__)`.
- The root logger is convenient for short scripts, but application code should prefer named loggers.
- Logger propagation means a record handled by `shop.orders` can also move upward to `shop` and then to the root logger.
- Duplicate log lines often happen when both a child logger and an ancestor logger have handlers while propagation remains enabled.
- A logger's effective level is the first explicit level found by walking up the logger hierarchy. If `shop.orders` has no level, it inherits from `shop`, then root.

## Configuring Logging

`basicConfig()` is fine for small scripts:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
```

Applications usually need more control. Prefer `logging.config.dictConfig()` because it keeps formatters, filters, handlers, logger names, levels, and propagation rules together in one explicit configuration.

```python
import logging.config

LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "standard": {
            "format": "%(asctime)s %(levelname)s %(name)s %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "standard",
            "level": "INFO",
        },
    },
    "loggers": {
        "shop": {
            "handlers": ["console"],
            "level": "INFO",
            "propagate": False,
        },
    },
}

logging.config.dictConfig(LOGGING)
logger = logging.getLogger("shop.orders")
```

## Useful Log Events

Log levels describe how important an event is:

- `debug`: detailed diagnostic data, usually disabled in production.
- `info`: normal business or lifecycle events.
- `warning`: something unexpected happened, but the program can continue.
- `error`: an operation failed.
- `exception`: like `error`, but includes the current exception traceback. Call it only inside an `except` block.
- `critical`: the application or a major subsystem may be unable to continue.

Prefer lazy `%s` formatting:

```python
logger.debug("Loaded %s rows from %s", row_count, path)
```

Avoid this in disabled debug logs:

```python
logger.debug(f"Loaded {expensive_count()} rows")
```

The f-string is evaluated before `logger.debug()` can decide whether the message is enabled. Lazy formatting defers interpolation until the record will actually be emitted.

Good log events are specific and useful:

```python
logger.info("order_accepted", extra={"order_id": order_id, "total": total})
logger.warning("order_validation_failed", extra={"reason": "empty_items"})
logger.exception("order_processing_failed", extra={"order_id": order_id})
```

Do not log secrets, passwords, tokens, full credit card numbers, private keys, or unnecessary personally identifiable information. Logs are often copied, indexed, retained, and read by many systems.

## Structured and Contextual Logging

Plain text logs are easy for humans to read. Structured logs are easier for machines to search and aggregate. Python's standard logging API lets you attach extra fields with `extra`.

```python
logger.info(
    "payment_authorized",
    extra={"order_id": "A100", "request_id": "req-123", "amount": 42.50},
)
```

Formatters can include those fields if they are always present. Filters can make that safer by adding default context to every record.

`contextvars` is useful when one request flows through many functions. A filter can read the current request ID from a context variable and add it to each `LogRecord`. This avoids passing `request_id` through every function just for logging.

Common contextual fields include:

- `request_id`: unique ID for one incoming request or job.
- `correlation_id`: ID shared across multiple services for the same user-visible operation.
- `order_id`, `user_id`, or `tenant_id`: business identifiers when safe to log.
- `duration_ms`: how long an operation took.
- `result`: success, validation_failed, error, or another bounded value.

Keep field values low-cardinality when they are used as metric labels. A label like `status="success"` is safe. A label like `user_email="..."` can create too many time series and may expose private data.

## Observability Signals

Observability is the ability to understand a system's behavior from the data it emits. The common signals are:

- Logs: timestamped events that describe what happened.
- Metrics: numeric measurements over time, such as request counts, error counts, queue depth, and latency.
- Traces: records of a request's path through a system.
- Spans: individual timed units of work inside a trace, such as `validate_order` or `charge_payment`.
- Correlation IDs: identifiers that connect logs, metrics, and traces for the same operation.

Logs explain individual events. Metrics show trends and alert conditions. Traces show where time was spent and how services interacted. Together, they let you move from "orders are failing" to "this request failed validation in the order service after 3 ms" or "payment calls are timing out after the gateway deploy."

## Mini Case Study: Order Processing

Imagine an order-processing service. Useful telemetry might include:

- A request ID created at the edge of the system.
- An `order_validation_failed` warning when an order has no items.
- An `order_processed` info event when the order succeeds.
- An `order_processing_failed` exception event when an unexpected error occurs.
- A duration field so slow requests can be found.
- Counter metrics for total orders, validation failures, successful orders, and exceptions.

A production service might export metrics with the Prometheus Python client and traces with OpenTelemetry. This lesson's worked answer uses only the standard library, but keeps the same shape: logs for events, counters for metrics, and request IDs for correlation.

## Common Mistakes

- Using the root logger everywhere instead of named module loggers.
- Calling configuration repeatedly and accidentally adding duplicate handlers.
- Leaving propagation enabled while handlers exist on both child and parent loggers.
- Logging secrets, tokens, passwords, or unnecessary PII.
- Swallowing exceptions after logging them, leaving the caller thinking the operation succeeded.
- Using f-strings or expensive function calls in disabled debug logs.
- Logging too much inside tight loops.
- Creating high-cardinality metric labels such as `order_id`, `request_id`, or `email`.
- Logging vague messages like `"failed"` without the operation, reason, or safe identifiers.

## Exercise

Write a runnable Python program that simulates order processing.

Requirements:

1. Configure logging with `logging.config.dictConfig()`.
2. Use a named logger, not the root logger directly.
3. Add a request ID to every log record with `contextvars` and a logging `Filter`.
4. Log a validation failure with `warning`.
5. Log a successful order with `info`.
6. Log an unexpected exception with `exception`.
7. Record operation duration in milliseconds.
8. Keep simple `Counter` metrics for attempts, successes, validation failures, and exceptions.
9. Do not log secrets or PII.

## Worked Answer

Save this as `order_observability_demo.py` and run it with `python order_observability_demo.py`.

```python
from __future__ import annotations

import contextvars
import logging
import logging.config
import time
import uuid
from collections import Counter
from dataclasses import dataclass


request_id_var = contextvars.ContextVar("request_id", default="-")
metrics = Counter()


class RequestContextFilter(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        record.request_id = request_id_var.get()
        return True


LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "filters": {
        "request_context": {
            "()": RequestContextFilter,
        },
    },
    "formatters": {
        "standard": {
            "format": (
                "%(asctime)s %(levelname)s %(name)s "
                "request_id=%(request_id)s %(message)s"
            ),
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "DEBUG",
            "formatter": "standard",
            "filters": ["request_context"],
        },
    },
    "loggers": {
        "shop": {
            "handlers": ["console"],
            "level": "DEBUG",
            "propagate": False,
        },
    },
}


logging.config.dictConfig(LOGGING)
logger = logging.getLogger("shop.orders")


@dataclass(frozen=True)
class Order:
    order_id: str
    items: list[str]
    fail_payment: bool = False


def validate_order(order: Order) -> bool:
    if not order.items:
        logger.warning(
            "order_validation_failed order_id=%s reason=%s",
            order.order_id,
            "empty_items",
        )
        metrics["orders.validation_failed"] += 1
        return False
    return True


def charge_payment(order: Order) -> None:
    if order.fail_payment:
        raise RuntimeError("payment gateway timeout")


def process_order(order: Order) -> bool:
    request_id = f"req-{uuid.uuid4().hex[:8]}"
    token = request_id_var.set(request_id)
    started = time.perf_counter()
    metrics["orders.attempted"] += 1

    try:
        logger.debug("order_processing_started order_id=%s", order.order_id)

        if not validate_order(order):
            duration_ms = (time.perf_counter() - started) * 1000
            logger.info(
                "order_rejected order_id=%s duration_ms=%.2f",
                order.order_id,
                duration_ms,
            )
            return False

        charge_payment(order)

        duration_ms = (time.perf_counter() - started) * 1000
        metrics["orders.succeeded"] += 1
        logger.info(
            "order_processed order_id=%s item_count=%s duration_ms=%.2f",
            order.order_id,
            len(order.items),
            duration_ms,
        )
        return True

    except Exception:
        duration_ms = (time.perf_counter() - started) * 1000
        metrics["orders.exceptions"] += 1
        logger.exception(
            "order_processing_failed order_id=%s duration_ms=%.2f",
            order.order_id,
            duration_ms,
        )
        raise

    finally:
        request_id_var.reset(token)


def main() -> None:
    orders = [
        Order(order_id="A100", items=["book", "pen"]),
        Order(order_id="A101", items=[]),
        Order(order_id="A102", items=["keyboard"], fail_payment=True),
    ]

    for order in orders:
        try:
            process_order(order)
        except RuntimeError:
            pass

    print("\nMetrics snapshot:")
    for name, value in sorted(metrics.items()):
        print(f"{name} {value}")


if __name__ == "__main__":
    main()
```

Notice the important choices:

- Logging is configured once, at application startup.
- Application code uses `logging.getLogger("shop.orders")`.
- The `RequestContextFilter` adds `request_id` to every emitted record.
- `logger.exception()` preserves the traceback and the exception is re-raised.
- Metrics use bounded names instead of labels containing `order_id` or `request_id`.
- Log messages avoid secrets and unnecessary personal data.

## Next Step

Return to the advanced README and connect this lesson to application configuration and deployment. Logs become much more useful when log level, format, destination, request IDs, and release version are configured consistently across environments.

## Sources Used

- Python Logging HOWTO: https://docs.python.org/3/howto/logging.html
- Python Logging Cookbook: https://docs.python.org/3/howto/logging-cookbook.html
- Python logging reference: https://docs.python.org/3/library/logging.html
- OpenTelemetry observability signals: https://opentelemetry.io/docs/concepts/signals/
- OpenTelemetry observability primer: https://opentelemetry.io/docs/concepts/observability-primer/
- Prometheus Python client Counter documentation: https://prometheus.github.io/client_python/instrumenting/counter/
