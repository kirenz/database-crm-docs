# CRM database — schema reference

Companion docs for the hosted teaching CRM database used in HdM's text-to-SQL
exercises. This repo is the **public schema reference**; the data generator
itself lives in a separate, private repository.

## Connect

```
host     = 49.12.14.207
port     = 55432
database = crm
user     = student
password = student
sslmode  = prefer
```

Read-only role. Works with any Postgres client (psql, DBeaver, TablePlus,
DataGrip, pgAdmin, …). Quick check from a terminal:

```bash
PGPASSWORD=student psql -h 49.12.14.207 -p 55432 -U student -d crm \
  -c "SELECT COUNT(*) FROM accounts;"
# → 1500
```

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
