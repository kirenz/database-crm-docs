# CRM database — schema reference & how to connect

Companion docs for the hosted teaching CRM database used in HdM's
text-to-SQL exercises. This repo is the **public schema reference**; the
data generator itself lives in a separate, private repository.

> **What is this?** A Postgres database with realistic CRM data —
> 1,500 companies, 7,000 sales opportunities, 40,000 activities, and more.
> It's hosted online so **you don't need to install Postgres yourself**.
> You just point a SQL client at it from your laptop and run queries.

## Connection details

The database is read-only and lives at a fixed address. Your instructor
sends you the connection values (host, port, user, password) separately by
email. The instructions below assume you have that email open in another
tab.

Wherever you see `<host>`, `<port>`, `<user>`, `<password>` below, paste in
the actual values from the email — don't type the angle brackets.

## How to connect from VS Code

You'll likely use **two** of the three options below: one for *exploring*
the data interactively (Option A or B), and one for *querying from Python*
in your exercise files (Option C).

If you've never used any of these tools before, that's fine — start with
**Option A** (graphical, point-and-click), and add Option C once you're
writing your first exercise.

---

### Option A — VS Code SQL extension *(recommended for exploring)*

A graphical Postgres client that lives inside VS Code. Best for browsing
tables, running ad-hoc queries, and getting a feel for the schema.

1. Open VS Code → click the **Extensions** icon in the left sidebar (the
   four-squares icon), or press `Ctrl+Shift+X` (Windows/Linux) /
   `Cmd+Shift+X` (macOS).
2. Search for **"PostgreSQL"** by *Chris Kolkman* and click *Install*.
   [Direct marketplace
   link.](https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-postgresql-client2)
3. After install, a new **database icon** appears in the left sidebar.
   Click it → **"+ Create Connection"** → choose **PostgreSQL**.
4. Fill in the values from your instructor's email (Host, Port, Username,
   Password, Database = `crm`). Click **Connect**.
5. You should see a tree of all 13 tables on the left. Right-click any
   table → **"Select Top 1000"** to peek at the data, or right-click the
   connection → **"New Query"** to write your own SQL.

If the connection hangs or fails, see [Troubleshooting](#troubleshooting)
below.

---

### Option B — Terminal with `psql` *(quick CLI queries)*

`psql` is the official command-line client for Postgres. It's the fastest
way to fire off a one-line query without leaving the terminal. You only
need the *client* — not a full Postgres server.

**Install `psql`** (one-time setup):

* **macOS:** [Homebrew](https://brew.sh/) is the standard package manager
  for macOS — it lets you install command-line tools with a single command.
  If you don't have it yet, follow the one-line install on the homepage.
  Then run these two commands in Terminal, **one at a time**:

  ```bash
  brew install libpq
  ```

  ```bash
  brew link --force libpq
  ```

  The first downloads `psql`; the second makes it findable on your `PATH`
  so VS Code's terminal can call it by name.

* **Windows:**
  Download the official
  [PostgreSQL installer](https://www.postgresql.org/download/windows/),
  run it, and during the component selection step pick **"Command Line
  Tools"** only — you can untick everything else, since the database itself
  is hosted (you don't need a local Postgres server). After the install
  finishes, close and reopen VS Code so it picks up the new `psql`
  command on your `PATH`.

If neither option appeals, just skip Option B — Options A and C cover
everything you need.

**Run the test query.** Open VS Code's integrated terminal
(menu *Terminal → New Terminal*, or press `` Ctrl+` ``  /  `` Cmd+` ``)
and run:

```bash
PGPASSWORD=<password> psql -h <host> -p <port> -U <user> -d crm \
  -c "SELECT COUNT(*) FROM accounts;"
```

Expected output:

```
 count
-------
  1500
(1 row)
```

---

### Option C — Python via `uv` *(for the actual exercise files)*

This is what you'll use *inside* your exercise scripts. You query the
database with [`psycopg`](https://www.psycopg.org/) — the standard Postgres
driver for Python.

If you don't have `uv` yet, follow
[**kirenz/uv-setup**](https://github.com/kirenz/uv-setup) first. `uv` is a
fast, modern Python project manager that handles installs, virtual
environments, and Python versions for you.

**One-shot test, no project setup needed:**

```bash
uv run --with "psycopg[binary]" python -c "
import psycopg
with psycopg.connect(
    'host=<host> port=<port> dbname=crm user=<user> password=<password>'
) as conn:
    print(conn.execute('SELECT COUNT(*) FROM accounts').fetchone())
"
```

Expected output: `(1500,)`. The `(value,)` syntax is just how Python
represents a single-row, single-column result — a *tuple of length one*.

**For your actual exercise project**, add the dependency once with `uv`,
then `import psycopg` like any other library:

```bash
cd /path/to/your/exercise/project
uv add "psycopg[binary]"
```

```python
# exercise.py
import psycopg

CONN = (
    "host=<host> port=<port> dbname=crm "
    "user=<user> password=<password>"
)

with psycopg.connect(CONN) as conn:
    rows = conn.execute("""
        SELECT industry, COUNT(*) AS n
        FROM accounts
        GROUP BY industry
        ORDER BY n DESC
    """).fetchall()
    for industry, n in rows:
        print(f"{industry:20s}  {n}")
```

Run it with `uv run python exercise.py`.

---

## Quick test — please do this once

Pick whichever option above looks easiest, fill in the values from your
instructor's email, and run the count query. If you see **`1500`**, you're
all set up. 🎉

## Troubleshooting

If something doesn't work, just message your instructor — but try these
checks first, since they cover 90% of issues.

* **"Connection refused" / "Operation timed out"** — most likely a
  firewall on your network. School and company networks often block
  non-standard ports. Quick test: try from a different network (e.g.
  your phone's hotspot). If it works there but not on the school WiFi,
  let your instructor know.
* **"FATAL: password authentication failed for user …"** — a typo. The
  password is case-sensitive. Re-paste it carefully from the email and
  make sure no trailing space sneaked in.
* **VS Code extension hangs forever on "Connecting…"** — check Host and
  Port are correct, and that you picked **PostgreSQL** (not MySQL or
  another DB) when creating the connection.
* **The Mermaid diagram on `erd.md` looks like raw text** — view it on
  GitHub directly (the link below), not a local preview. GitHub renders
  Mermaid automatically; most local Markdown previewers don't.

When in doubt: copy the exact error message and reply to the instructor's
email. Don't lose hours fighting setup — that's not the interesting part.

---

## Schema

* **[`erd.md`](erd.md)** — visual entity-relationship diagram + per-table
  column reference. Open it on GitHub to see the diagram render.

The 13 tables mirror the Salesforce standard-object model:

```
users (SDR / AE / Manager)
  └── accounts
       ├── contacts
       ├── opportunities
       │    └── opportunity_line_items → products
       ├── activities (polymorphic: account | opportunity | lead)
       └── cases
leads (40% converted → linked back to account + opportunity)
campaigns
  └── campaign_members
products
  └── pricebook_entries → pricebooks
```

## Data conventions

* Time is **frozen at 2026-04-24 12:00 UTC**. For time-relative queries
  ("last 6 months", "this quarter") use the SQL function `crm_now()` instead
  of `NOW()` — answers stay stable regardless of when you run them.

  ```sql
  SELECT COUNT(*) FROM activities
  WHERE completed_at >= crm_now() - INTERVAL '6 months';
  ```

  Using `NOW()` will return empty results for most date filters, which is
  confusing — `crm_now()` fixes that.

* The polymorphic `activities` table has three nullable FKs
  (`account_id`, `opportunity_id`, `lead_id`) plus a `related_to_type`
  discriminator. **Use the convenience views**, not raw filters:

  | View                       | What it returns |
  | -------------------------- | --------------- |
  | `opportunity_activities`   | activities where `related_to_type = 'opportunity'` |
  | `lead_activities`          | activities where `related_to_type = 'lead'` |
  | `account_all_activities`   | every activity *about* an account — direct + opportunity-routed + converted-lead-routed. Has a `via` column. |

  Naive query like `SELECT COUNT(*) FROM activities WHERE account_id IS NOT NULL`
  silently misses opportunity- and lead-scoped touches that also belong to
  the account.

## Row counts

| Table                   | Rows     |
| ----------------------- | -------- |
| users                   | 50       |
| accounts                | 1 500    |
| contacts                | ~4 500   |
| leads                   | 3 000    |
| opportunities           | 7 000    |
| products                | 200      |
| pricebooks              | 2        |
| pricebook_entries       | 400      |
| opportunity_line_items  | ~15 000  |
| campaigns               | 30       |
| campaign_members        | 5 000    |
| activities              | 40 000   |
| cases                   | 500      |
