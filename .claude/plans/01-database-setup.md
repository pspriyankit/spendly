# Plan: Database Setup (Step 1)

## Context

Spendly currently has an empty `database/db.py` stub and `app.py` has no DB initialization. All future features (auth, expenses, profile) depend on this data layer. This step implements the three core DB helpers and wires them into app startup.

---

## Files to Change

- `database/db.py` — implement all three functions
- `app.py` — import helpers and call `init_db()` + `seed_db()` on startup

## Files to Create

- None

---

## Implementation Steps

### 1. `database/db.py`

**Imports:**
```python
import sqlite3
from werkzeug.security import generate_password_hash
```

**`get_db()`**
- Opens `spendly.db` in the project root (relative path so it lands next to `app.py`)
- Sets `conn.row_factory = sqlite3.Row`
- Executes `PRAGMA foreign_keys = ON`
- Returns the connection

**`init_db()`**
- Calls `get_db()` to get a connection
- Creates `users` table with `CREATE TABLE IF NOT EXISTS`:
  - `id INTEGER PRIMARY KEY AUTOINCREMENT`
  - `name TEXT NOT NULL`
  - `email TEXT UNIQUE NOT NULL`
  - `password_hash TEXT NOT NULL`
  - `created_at TEXT DEFAULT (datetime('now'))`
- Creates `expenses` table with `CREATE TABLE IF NOT EXISTS`:
  - `id INTEGER PRIMARY KEY AUTOINCREMENT`
  - `user_id INTEGER NOT NULL REFERENCES users(id)`
  - `amount REAL NOT NULL`
  - `category TEXT NOT NULL`
  - `date TEXT NOT NULL`
  - `description TEXT`
  - `created_at TEXT DEFAULT (datetime('now'))`
- Commits and closes connection

**`seed_db()`**
- Calls `get_db()`
- Checks `SELECT COUNT(*) FROM users` — if > 0, return early
- Inserts demo user:
  - name: `Demo User`, email: `demo@spendly.com`
  - password_hash: `generate_password_hash("demo123")`
  - Gets the new user's `lastrowid`
- Inserts 8 sample expenses linked to that `user_id`, covering all 7 categories (Food, Transport, Bills, Health, Entertainment, Shopping, Other) with YYYY-MM-DD dates spread across the current month and `amount` as REAL
- Uses parameterized queries (`?` placeholders) throughout
- Commits and closes connection

---

### 2. `app.py`

Add imports at the top:
```python
from database.db import get_db, init_db, seed_db
```

Add startup initialization after `app = Flask(__name__)`:
```python
with app.app_context():
    init_db()
    seed_db()
```

No routes change.

---

## Categories (fixed list)

`Food`, `Transport`, `Bills`, `Health`, `Entertainment`, `Shopping`, `Other`

---

## Verification

1. Run `python app.py` — should start on port 5001 with no errors
2. A `spendly.db` file should appear in the project root
3. Open a Python shell and verify:
   ```python
   from database.db import get_db
   db = get_db()
   print([dict(r) for r in db.execute("SELECT * FROM users").fetchall()])
   print(db.execute("SELECT COUNT(*) FROM expenses").fetchone()[0])  # should be 8
   ```
4. Run `python app.py` a second time — no duplicate seed data (still 1 user, 8 expenses)
5. Run `pytest` — all existing tests pass
