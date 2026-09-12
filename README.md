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
│   ├── lib.alya            # Public API facade
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

---

## 📦 Installation

Add `sqlite` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
sqlite = { git = "https://github.com/alya-lang/sqlite", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alyac add sqlite --git https://github.com/alya-lang/sqlite --branch main
alyac install
```

---

## 🚀 Quick Start

```alya
import "sqlite"

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
```

---

## 📚 API Reference

### Database Lifecycle
- `sqlite::open(path: str) -> Database`: Opens or creates a file-based SQLite database.
- `sqlite::open_memory() -> Database`: Opens a private, in-memory SQLite database (`:memory:`).
- `sqlite::close(db: Database)`: Closes database handle.
- `sqlite::error_message(db: Database) -> str`: Returns latest error message from engine.
- `sqlite::version() -> str`: Returns SQLite library version (e.g. `"3.53.4"`).
- `sqlite::source_id() -> str`: Returns SQLite source code identifier.

### Query Execution
- `sqlite::execute(db: Database, sql: str) -> i32`: Executes statement and returns affected row count.
- `sqlite::query(db: Database, sql: str) -> Array`: Executes `SELECT` and returns array of row maps.
- `sqlite::query_scalar(db: Database, sql: str) -> value`: Executes query and returns single scalar value.
- `sqlite::table_exists(db: Database, table_name: str) -> bool`: Checks if table exists in schema.
- `sqlite::last_insert_id(db: Database) -> i64`: Returns `ROWID` of last inserted row.
- `sqlite::changes(db: Database) -> i32`: Returns number of rows modified by last query.

### Row Value Helpers
- `sqlite::row_str(row: Map, key: str) -> str`: Extracts string value from row safely.
- `sqlite::row_int(row: Map, key: str) -> i32`: Extracts integer value from row.
- `sqlite::row_float(row: Map, key: str) -> f64`: Extracts numeric float value from row.

---

## ⚡ Performance Benchmarks

Measured on Windows 11 / x86_64:

```text
=== Benchmark Suite: SQLite3 Performance Benchmarks ===
  * 5,000 In-Memory Transactional Inserts: 5000 iters in 12 ms (~2400 ns/op | 416,666 ops/sec)
  * 100x Query (500 rows each): 100 iters in 46 ms (~460 µs/op | 2,173 ops/sec)
```

---

## 📄 License

MIT License © [Alya Language](https://github.com/alya-lang)