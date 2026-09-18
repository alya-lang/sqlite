# sqlite

[![CI](https://github.com/alya-lang/sqlite/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/sqlite/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/sqlite?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fsqlite%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fsqlite%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

High-performance, idiomatic SQLite3 relational database bindings for Alya via native C FFI (`extern "C"`).

---

## 🌟 Features

- ⚡ **Full C Speed**: Direct zero-overhead FFI bindings to the official SQLite3 engine (>400,000 inserts/sec).
- 📦 **Zero External Dependencies**: Official SQLite3 C engine bundled directly (`c/sqlite3.c`). Automatically compiled and cached with zero DLLs, `.so`, or `.dylib` needed!
- 💾 **File & In-Memory Databases**: Connect to persistent `.db` files or lightning-fast transient `:memory:` databases.
- 🛡️ **Prepared Statements**: Safe SQL query parsing and step-by-step row iteration.
- 🗺️ **Dynamic Maps**: Automatic column name mapping into native Alya maps (`row["column_name"]`).
- 🔄 **Transactions & Changes**: Full ACID transactions (`BEGIN`, `COMMIT`, `ROLLBACK`), `changes()`, and `last_insert_id()`.
- 🧩 **Zero Compiler Bloat**: Pure package implementation without hacking compiler internals.

---

## 📁 Project Architecture

```text
sqlite/
├── alya.toml               # Package manifest with [build] c-sources
├── c/                      # Bundled SQLite3 C Amalgamation
│   ├── sqlite3.c           # Full official SQLite3 engine source
│   └── sqlite3.h           # SQLite3 C headers
├── src/
│   ├── lib.alya            # Public API facade & top-level re-exports
│   ├── types.alya          # SQLite constants & Database struct
│   ├── ffi.alya            # Native extern "C" declarations
│   └── core/
│       └── database.alya   # Engine lifecycle, query executor & row mapper
├── examples/
│   └── demo.alya           # Full CRUD & SQL JOIN demonstration
├── tests/
│   └── test_basic.alya     # Automated test suite
└── benches/
    └── bench_basic.alya    # Micro-benchmarks (>400k ops/s)
```

> [!NOTE]
> The SQLite3 C engine source is declared in `alya.toml` under `[build]`. During compilation, `alya` automatically compiles `c/sqlite3.c` into an object file and caches it in `~/.alya/c_obj`, guaranteeing zero runtime installation requirements across Linux, macOS, and Windows.

---

## 📦 Installation

Add `sqlite` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
sqlite = { git = "https://github.com/alya-lang/sqlite", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add sqlite --git https://github.com/alya-lang/sqlite --branch main
alya install
```

---

## 🚀 Quick Start

```alya
import "sqlite"

function main()
    # 1. Open database (file or in-memory)
    let db = sqlite::open("app.db")
    # or: let db = sqlite::open_memory()

    # 2. Execute DDL statements
    sqlite::execute(db, "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, score REAL);")

    # 3. Insert records
    sqlite::execute(db, "INSERT INTO users (name, score) VALUES ('Alice', 95.5);")
    let alice_id = sqlite::last_insert_id(db)

    sqlite::execute(db, "INSERT INTO users (name, score) VALUES ('Bob', 88.0);")

    # 4. Scalar query
    let total = sqlite::query_scalar(db, "SELECT count(*) FROM users;")
    say "Total registered users: " + str(total)

    # 5. Query rows
    let rows = sqlite::query(db, "SELECT id, name, score FROM users ORDER BY score DESC;")
    for r in rows
        let id = sqlite::row_int(r, "id")
        let name = sqlite::row_str(r, "name")
        let score = sqlite::row_str(r, "score")
        say "User #" + str(id) + ": " + name + " -> Score: " + score
    end

    # 6. Close database
    sqlite::close(db)
end

main()
```

---

## 📖 API Reference

### Database Lifecycle
| Function | Arguments | Returns | Description |
|---|---|---|---|
| `open(path)` | `path: str` | `Database` | Opens or creates a file-based SQLite database. |
| `open_memory()` | none | `Database` | Opens a private, in-memory SQLite database (`:memory:`). |
| `close(db)` | `db: Database` | `void` | Closes the open database connection and frees resources. |
| `error_message(db)` | `db: Database` | `str` | Returns the most recent error message produced by the engine. |
| `version()` | none | `str` | Returns the underlying SQLite3 library version string (e.g. `"3.53.4"`). |
| `source_id()` | none | `str` | Returns the SQLite3 engine source code identifier and timestamp. |

### Query Execution
| Function | Arguments | Returns | Description |
|---|---|---|---|
| `execute(db, sql)` | `db: Database, sql: str` | `i32` | Executes an SQL command (DDL/DML) and returns affected row count. |
| `query(db, sql)` | `db: Database, sql: str` | `Array` | Executes a `SELECT` query and returns an array of row map dictionaries. |
| `query_scalar(db, sql)` | `db: Database, sql: str` | `value` | Executes a query and returns the first column of the first row. |
| `table_exists(db, name)` | `db: Database, name: str` | `bool` | Returns `true` if the specified table exists in the database schema. |
| `last_insert_id(db)` | `db: Database` | `i64` | Returns the `ROWID` of the most recently inserted row. |
| `changes(db)` | `db: Database` | `i32` | Returns the number of rows modified, inserted, or deleted by the last statement. |

### Row Value Helpers
| Function | Arguments | Returns | Description |
|---|---|---|---|
| `row_str(row, key)` | `row: Map, key: str` | `str` | Safely extracts a string value from a query row map. |
| `row_int(row, key)` | `row: Map, key: str` | `i32` | Safely extracts an integer value from a query row map. |
| `row_float(row, key)` | `row: Map, key: str` | `f64` | Safely extracts a floating-point numeric value from a query row map. |

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya`:

```bash
alya test
# or
alya run tests/test_basic.alya
```

Run performance micro-benchmarks:

```bash
alya run benches/bench_basic.alya
```

Run runnable usage example:

```bash
alya run examples/demo.alya
```

### Benchmark Results (Windows 11 / x86_64)

```text
=== Benchmark Suite: SQLite3 Performance Benchmarks ===
  * 5,000 In-Memory Transactional Inserts: 5000 iters in 12 ms (~2400 ns/op | 416,666 ops/sec)
  * 100x Query (500 rows each): 100 iters in 46 ms (~460 µs/op | 2,173 ops/sec)
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.