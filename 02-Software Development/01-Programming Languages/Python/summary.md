# Python — Compact Reference

## Basics
- **syntax**: Indentation-based blocks (4 spaces). `#` comments. `"""..."""` docstrings. Compiled to `.pyc` bytecode, then interpreted.
- **variables**: Dynamic typing. `type()` to inspect. `is` checks identity, `==` checks value equality.
- **data types**: `int` (arbitrary precision), `float`, `str`, `bool`, `None`. f-strings since 3.6: `f"{name=}"`.
- **conditionals**: `if/elif/else`. Ternary: `x if c else y`. `match/case` (3.10+) for structural pattern matching.
- **loops**: `for x in iterable`, `while cond`. `range()` lazy. `enumerate()` for index+value. Loop `else` fires if no `break`.
- **exceptions**: `try/except/else/finally`. `except*` for `ExceptionGroup` (3.11+). `raise` to re-raise.
- **functions**: `def f(*args, **kwargs) -> T:`. `lambda x: expr` (single expression only). `global`/`nonlocal` for scope.

## Collections
- **list**: `[]`, mutable, ordered, O(1) append, O(n) insert at head. Growth: `~12.5%` over-allocation.
- **tuple**: `()`, immutable, hashable, faster. Single-element: `(x,)`.
- **set**: `{}`, unordered, unique elements, O(1) membership. `| & - ^` operators. `frozenset` for immutable.
- **dict**: `{k: v}`, ordered since 3.7, O(1) lookup. Compact impl (3.6+): sparse indices + dense entries. Merge: `d1 | d2` (3.9+).

## Data Structures
- **stack**: `list.append` + `list.pop` — O(1).
- **queue**: `collections.deque` — `append`/`popleft` O(1). Never `list.pop(0)`.
- **heap**: `heapq` — min-heap, O(log n) push/pop, O(n) heapify. Negate values for max-heap.
- **hash table**: `dict`/`set` use open addressing (not chaining). Keys must be hashable (immutable).
- **BST**: No built-in. Implement `TreeNode` manually. O(h) ops, O(n) worst case if skewed.
- **sorting**: Timsort — O(n log n), stable, O(n) best. `list.sort()` in-place, `sorted()` out-of-place. `key=` for custom.

## Modules & Packages
- **import**: `import mod`, `from mod import item`, `import mod as alias`. `from mod import *` — avoid.
- **stdlib**: `os`, `sys`, `pathlib` (modern), `json`, `datetime`, `collections` (Counter, defaultdict, deque), `itertools`, `functools`.
- **custom package**: Directory with `__init__.py`. `__name__ == "__main__"` guard for direct execution. `sys.path` search order.

## Advanced Functional
- **lambda**: Anonymous, single expression. PEP 8: don't assign to variable; use `def`. Ternary allowed: `lambda x: "E" if x%2==0 else "O"`.
- **decorator**: `@decorator` wraps function. `@functools.wraps` preserves metadata. Stacked bottom-up. ParamSpec for typed decorators.
- **iterator**: Implements `__iter__` + `__next__`. Exhausted after one pass. `iter(obj)`, `next(it)`.
- **list comprehension**: `[expr for x in iter if cond]`. C-level speed. If/else: `[a if c else b for x in iter]`. Nested for flattening.
- **generator**: `(expr for x in iter)` — lazy, constant memory. `yield` for generator functions. `yield from` delegates. `itertools.islice` to slice.
- **regex**: `re` module. `search()` vs `match()` (match = start only). Raw strings: `r"\d+"`. Groups with `()`. `re.compile` for reuse.

## Package Management
- **PyPI**: Official package index. Wheels (`.whl`) = pre-built. `twine` for upload. TestPyPI for testing.
- **pip**: `pip install`, `pip freeze > requirements.txt`. No true lock file. Editable install: `pip install -e .`.
- **conda**: Cross-language, binary packages. `conda create -n env python=3.x`. `environment.yml`. Conda-forge channel.
- **uv**: Rust-based, 10-100x pip speed. `uv init`, `uv add`, `uv sync`, `uv run`. Manages Python versions + lock file. Workspaces.
- **poetry**: `pyproject.toml` + `poetry.lock`. PubGrub resolver. `poetry add`, `poetry install`, `poetry shell`. Pre-uv gold standard.
- **pyproject.toml**: PEP 621 standard. `[build-system]`, `[project]`, `[tool.*]` sections. Replaces `setup.py`/`setup.cfg`/`requirements.txt`.

## Paradigms
- **OOP**: Classes, inheritance, polymorphism (duck typing: "if it quacks...").
- **FP**: First-class functions, `map`/`filter`/`reduce`, lambdas, immutability where possible.
- **Pythonic hybrid**: List comprehensions over `map`/`filter`. Simple functions over deep class hierarchies.
- **context manager**: `with` statement. `__enter__` + `__exit__`. `@contextmanager` from `contextlib`. Guarantees cleanup.

## Common Third-Party Packages
- **Web**: `requests` (HTTP), `httpx` (async HTTP). **Data**: `numpy`, `pandas`, `pydantic`.
- **Testing**: `pytest`, `coverage`. **Tooling**: `ruff`, `black`, `mypy`. **Utils**: `tqdm`, `python-dotenv`.

## OOP
- **classes**: `class C:`. `__init__(self)`. Instance vs class variables. `@dataclass` (3.7+) auto-generates `__init__`/`__repr__`/`__eq__`.
- **inheritance**: Single + multiple. `super()` follows MRO. Mixins for reusable behaviors.
- **MRO**: C3 linearization. `ClassName.mro()`. Solves diamond problem — common base visited once, after descendants.
- **ABC**: `from abc import ABC, abstractmethod`. Cannot instantiate abstract class.
- **methods**: Instance (`self`), Class (`@classmethod`, `cls`), Static (`@staticmethod`, no implicit arg).
- **dunder**: `__str__` (user) vs `__repr__` (debug). `__len__`, `__eq__`, `__call__`, `__add__`, `__lt__`. Operator overloading.

## Environments
- **venv**: Built-in since 3.3. `python -m venv .venv`. Isolates `site-packages`.
- **virtualenv**: Third-party, faster, supports different Python versions.
- **pyenv**: Manages Python versions via shims. `.python-version` per project. Precedence: shell > local > global > system.
- **pipenv**: `Pipfile` + `Pipfile.lock`. `pipenv install`, `pipenv shell`. Largely superseded by Poetry/uv.
- **poetry**: `pyproject.toml` + `poetry.lock`. Auto virtualenv in cache or `in-project`. `poetry run`, `poetry shell`.

## Static Typing
- **typing**: `list[int]`, `Optional[T]`/`T | None`, `Union`/`|` (3.10+). `TypeVar`, `Callable`, `Protocol` (static duck typing).
- **mypy**: Python-based. `--strict`. `# type: ignore[code]`. `.pyi` stubs, `types-*` packages.
- **pyright**: TypeScript/Node.js, 3-5x faster. Microsoft, Pylance. Modes: `off`/`basic`/`strict`.
- **pyre**: OCaml, Meta. Daemon + Watchman. Pysa = taint analysis for security.
- **pydantic**: Runtime validation + coercion. `BaseModel` + type hints. `@field_validator`. v2 core in Rust.

## Concurrency
- **GIL**: CPython mutex — one thread executes bytecode at a time. Protects reference counting. Released during I/O. PEP 703 (3.13+): optional no-GIL.
- **threading**: Best for I/O-bound. `Thread(target=fn).start()`. `Lock`, `RLock`, `Semaphore`, `Event`. `counter += 1` not atomic.
- **multiprocessing**: True parallelism, bypasses GIL. `Pool.map()`, `Queue`/`Pipe` for IPC. `Manager` for shared objects. `if __name__ == "__main__"` mandatory.
- **asyncio**: Event loop, single-threaded cooperative. `async def`/`await`. `create_task()` for concurrency. `gather()` for parallel await. Never block the loop.
- **aiohttp**: Async HTTP client/server. `ClientSession` for connection pooling. WebSocket support. Middlewares + signals.
## Code Quality
- **black**: Uncompromising formatter. Default line length 88. AST safety check. `black --check` for CI.
- **ruff**: Rust-based linter + formatter. 10-100x faster. Replaces Flake8 + Black + isort + pyupgrade.
- **yapf**: Google's formatter. Cost-based algorithm. Highly configurable. `based_on_style = google`.
- **sphinx**: Documentation generator. reST or MyST (Markdown). `autodoc` from docstrings. Themes: Furo, RTD.

## Testing
- **unittest**: Built-in xUnit. `TestCase`, `setUp`/`tearDown`, `assertEqual`. `unittest.mock` for patching.
- **pytest**: Industry standard. Function-based, plain `assert`. Fixtures (dependency injection, `yield` for teardown). `@parametrize`. `conftest.py`. 800+ plugins.
- **tox**: Multi-environment test runner. `tox.ini` with `envlist`. CI frontend. `tox -e py310 -- -v`.
- **doctest**: Executable docstrings via `>>>`. Good for examples, brittle for complex logic.
- **nose**: Deprecated. `nose2` successor, but `pytest` won. Legacy only.
## Frameworks

| Framework | Type | Key Feature |
|-----------|------|-------------|
| **FastAPI** | Sync + Async | Pydantic + Starlette, auto OpenAPI, DI, modern default |
| **Django** | Sync + Async | Batteries-included, ORM, admin, MVT, Channels for WS |
| **Flask** | Sync + Async (2.0+) | Microframework, Blueprints, extensions, flexible |
| **aiohttp** | Async | Client + server, `ClientSession`, WebSockets, middlewares |
| **Sanic** | Async | Flask-like, `uvloop`, fastest Python framework |
| **Tornado** | Async | Native WebSockets, `IOLoop`, solved C10k, FriendFeed |
| **Pyramid** | Sync | URL dispatch + traversal, ACLs, "start small, finish big" |
| **gevent** | Async | Greenlets (monkey-patching), sync-looking async code |
| **Plotly Dash** | Sync | React-based dashboards, layout + callbacks, Flask under |

## Python Rules
1. Use `pathlib`, not `os.path`.
2. Prefer list/dict comprehensions over `map`/`filter`.
3. Always `@functools.wraps` on decorators.
4. Never mutate default arguments: `def f(x=[])` → `def f(x=None)`.
5. Use `with` for resources (files, locks, connections).
6. `is` for `None`/`True`/`False`; `==` for value equality.
7. Async: never `time.sleep()` in coroutine; use `await asyncio.sleep()`.
8. CPU-bound → `multiprocessing`. I/O-bound → `asyncio` or `threading`.
9. Prefer `ruff` over separate Flake8 + Black + isort.
10. `pyproject.toml` is the single source of truth. Use `uv` or `poetry`, not raw `pip`.
