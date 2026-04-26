# CRM database — schema reference & how to connect

Companion docs for the hosted teaching CRM database used in HdM's
text-to-SQL exercises. This repo is the **public schema reference**; the
data generator itself lives in a separate, private repository.

## Connect

The database is hosted and read-only. Connection details (host, port, user,
password) are shared separately by the instructor.

## Three ways to connect from VS Code

Use whichever feels most natural — they all hit the same database.

### 1. Terminal + `psql` (simplest)

If `psql` is installed (`brew install libpq` on macOS, `sudo apt install
postgresql-client` on Linux/WSL):

```bash
PGPASSWORD=<password> psql -h <host> -p <port> -U <user> -d crm \
  -c "SELECT COUNT(*) FROM accounts;"
```

Expected output: **1500**.

### 2. Python via `uv` (no separate install)

For querying from a script or notebook:

```bash
uv run --with "psycopg[binary]" python -c "
import psycopg
with psycopg.connect(
    'host=<host> port=<port> dbname=crm user=<user> password=<password>'
) as conn:
    print(conn.execute('SELECT COUNT(*) FROM accounts').fetchone())
"
```

Expected output: `(1500,)`

For your actual exercise files, add the dependency once with
`uv add "psycopg[binary]"` and `import psycopg` as usual.

### 3. VS Code SQL extension

Install **"PostgreSQL"** by Chris Kolkman from the VS Code Marketplace,
then create a new connection with the values from the instructor. You get a
sidebar tree of all tables, a query editor with autocomplete, and inline
result tables — handy when you're exploring the schema.

## Quick test

Pick any of the three methods above and run the count query. If you see
**1500**, you're set. If anything fails — connection refused, timeout,
weird error — message the instructor and we'll debug together. Don't stay
stuck on setup.

## Schema

* **[`erd.md`](erd.md)** — visual entity-relationship diagram + per-table
  column reference. Mermaid renders directly on GitHub.

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
