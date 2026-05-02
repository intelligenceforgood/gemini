---
applyTo: "**/*.py"
---

# Python Standards

- **Type hints:** Required on all functions — parameters + return type.
- **Docstrings:** Google-style. Required on public functions, classes, modules. Private helpers (`_name`) get at least a one-liner.
- **Formatter:** Black at 120 chars. isort (profile = black).
- **Naming:** `snake_case` functions/variables/methods/files. `PascalCase` classes. `UPPER_SNAKE_CASE` constants. Singular nouns for constants and config keys.
- **Pydantic:** `snake_case` fields in Python. Use `CamelModel` with `alias_generator = to_camel` for JSON output — never write manual casing functions.
- **Imports:** Absolute preferred (`from i4g.store.review_store import ReviewStore`). Relative only within same sub-package. No wildcards.
- **Error handling:** Catch specific exceptions. Never bare `except:`. Log with `logger.exception()`.
- **Datetime:** Use `datetime.now(datetime.UTC)`, never `datetime.utcnow()`.
- **Logging:** `logger = logging.getLogger(__name__)` per module. Structured key-value pairs.
- **Settings:** Always via `get_settings()`, not hard-coded values. Override via env vars, not code.
- **Stores:** Use factories (e.g., `factories.py`) — do not instantiate stores directly.
- **TYPE_CHECKING guard:** For expensive runtime-only imports used solely as types (e.g., Playwright `Page`), guard with `if TYPE_CHECKING:`.

# Ruff Pitfalls

- **B008 — Typer `Option()`/`Argument()` in defaults.** Typer's documented pattern `def cmd(name: str = typer.Option(...))` triggers B008. Don't refactor — add `"src/*/cli/*.py" = ["B008"]` to `[tool.ruff.lint.per-file-ignores]` in pyproject.toml.
- **UP038 — `isinstance` with tuple of types.** `isinstance(x, (int, float))` → `isinstance(x, int | float)`. Use union syntax on Python 3.10+.
- **UP042 — `str, Enum` base classes.** `class Foo(str, Enum)` → `class Foo(StrEnum)` using `from enum import StrEnum` (Python 3.11+).
- **SIM105 — try/except/pass.** Replace with `contextlib.suppress(ExceptionType)`. Remember to `import contextlib`.
- **SIM102 — nested `if` statements.** Merge `if a: if b:` into `if a and b:` when the outer `if` has no `else`.
- **F841 — unused variable assignment.** Remove the assignment or prefix with `_` if the call has side effects (e.g., `_client = SomeClass()`).
- **UP037 — quoted type annotations.** Remove quotes from annotations when `from __future__ import annotations` is not needed (Python 3.11+ with union syntax).
- **E501 — line too long.** Break long `if` conditions into parenthesized multi-line format. Break long strings with implicit concatenation.
- **Redundant except clauses.** `except (SpecificError, Exception)` is redundant — `Exception` subsumes `SpecificError`. Catch the specific exception only.