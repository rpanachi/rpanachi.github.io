---
layout: post
title: How to not destroy your production database
date: 2026-06-12 00:00:00
categories: [database, sql, migrations, best practices]
excerpt: "Best practices and tips for safe data migrations"
archive: true
---

> "Data is a precious thing and will last longer than the systems themselves."<br/>
> — Tim Berners-Lee

## TL;DR

Keep data changes out of migration files. Make every data script idempotent, batched, and dry-run by default — and never trust an `ALTER` you haven't read. A bad deploy can be reverted; an `UPDATE` without a `WHERE` clause cannot.

Over the years I've collected a fair share of scars — what worked, what didn't, and the mistakes that were entirely avoidable. What follows are the practices that came out of them: the kind that keep you from being paged at 3 a.m. or frantically restoring a database backup while production is on fire.

The rules below are stack-agnostic. The examples use plain SQL (MySQL/PostgreSQL flavors noted where they differ) and pseudocode — adapt the syntax to your framework, keep the rules.

## Keep migrations schema-only

Migration files exist to evolve the schema. **They must not contain `INSERT`, `UPDATE`, `DELETE`, backfills or seeding of reference rows.** This is the anti-pattern:

```sql
-- migration: add_status_to_orders
ALTER TABLE orders ADD COLUMN status VARCHAR(20) NULL;

-- BAD: data change smuggled into a schema migration
UPDATE orders SET status = 'pending' WHERE status IS NULL;
```

Looks harmless, works fine in development, and fails in production in several independent ways:

* **Your migration may not run where you think it does.** Branching platforms (PlanetScale, for example) apply schema changes on an isolated copy and merge only the schema back — data written by the migration is silently lost.
* **It couples a slow operation to a fast one.** A backfill over millions of rows can take hours, hold locks, stall replication, or get killed by the pipeline's timeout halfway through.
* **A failure mid-data-step corrupts your migration state.** MySQL implicitly commits around DDL, so the `ALTER` can't be rolled back when the `UPDATE` dies. PostgreSQL's DDL is transactional — but only if your migration tool actually wraps the migration in one, and a long backfill should never live inside one transaction anyway.
* **It can't be batched, throttled, resumed or dry-run.** All the safety tooling from the next two sections is unavailable inside a migration file.

The correct shape is a three-step pattern you'll see again in this post: **add nullable → backfill via script → tighten**.

```sql
-- migration 1: schema only
ALTER TABLE orders ADD COLUMN status VARCHAR(20) NULL;

-- data change: separate idempotent script (next section)

-- migration 2, days later, after the backfill is verified:
ALTER TABLE orders MODIFY COLUMN status VARCHAR(20) NOT NULL DEFAULT 'pending';  -- MySQL
```

The column `DEFAULT` handles *new* rows from the moment the schema lands; the backfill handles *existing* rows; the tightening migration runs only when both are true.

And no, lookup tables (countries, roles, currencies) are not an exception. Seeders handle fresh environments; a one-time idempotent script introduces them into existing ones. Both are versioned code; neither lives inside a migration.

## Make data changes idempotent, rerunnable scripts

Any change to data — backfill, correction, deletion — is a standalone command in your framework's task runner, versioned in the repo and code-reviewed like everything else. Two things, no exceptions:

### It must be safe to run twice

Retries, partial failures and overlapping executions happen. Pick an idempotency mechanism and test the re-run: execute it twice in staging; the second run must change zero rows.

Naturally idempotent predicates — the `WHERE` selects only un-migrated rows, so each run processes the remainder:

```sql
UPDATE orders SET status = 'pending' WHERE status IS NULL;
```

Upserts for inserting reference rows (requires a unique key on the natural identifier):

```sql
-- PostgreSQL
INSERT INTO roles (key, name) VALUES ('auditor', 'Auditor')
ON CONFLICT (key) DO UPDATE SET name = EXCLUDED.name;

-- MySQL 8.0.19+
INSERT INTO roles (`key`, name) VALUES ('auditor', 'Auditor') AS new
ON DUPLICATE KEY UPDATE name = new.name;
```

An execution ledger for scripts whose effect can't be detected from the data itself: a tiny table recording which named scripts already ran; the script checks it and exits early.

### It must be batched

A single `UPDATE` over millions of rows holds locks for the full duration, bloats the undo log and stalls replicas. Process in batches over an indexed key:

```text
BATCH = 1000
last_id = checkpoint.load() or 0

loop:
    ids = sql("SELECT id FROM orders
               WHERE id > :last_id AND status IS NULL
               ORDER BY id LIMIT :batch", last_id, BATCH)
    if ids is empty: break

    within transaction:
        sql("UPDATE orders SET status = 'pending'
             WHERE id IN (:ids)
               AND status IS NULL", ids)   # repeat the predicate on the write!

    last_id = max(ids)
    checkpoint.save(last_id)               # crash here → resume, not restart
    progress.log(last_id, rows=len(ids))
    sleep(throttle)
```

Why each piece matters:

* **Repeat the predicate on the write.** Live traffic may have set a row to `'paid'` between your `SELECT` and your `UPDATE` — without `AND status IS NULL` on the write, you clobber it back.
* **Keyset pagination, never `OFFSET`.** `OFFSET` re-scans skipped rows on every batch and skips or repeats rows when the data shifts underneath it — which is exactly what a backfill does.
* **One transaction per batch.** A multi-hour transaction is an outage waiting for a deadlock.
* **Checkpoint, throttle, log progress.** A crash at row 4,000,000 resumes instead of restarting; replicas keep up; "is it stuck?" becomes a glance instead of a guess.

Run it manually, per environment, after the schema migration has landed — on purpose, watching it run. Never as a hidden deploy side effect.

## Default to dry-run

Every script that deletes or bulk-modifies data supports a dry-run mode — and dry-run is the default. Mutation requires an explicit opt-in:

```text
purge_abandoned_carts [--execute] [--max-affected=N]

  affected = count rows matching the predicate
  print("would delete {affected} rows; sample:")
  print(first 10 matching rows)

  if not --execute:
      print("DRY RUN — re-run with --execute to apply"); exit

  if affected > --max-affected (default 50k):
      abort("affected rows exceed safety ceiling")

  ... batched deletion as in the previous section ...
```

* **Safe by default.** The flag enables danger (`--execute`), never disables it. Forgetting a flag should always land you on the safe side.
* **Counts AND a sample.** The count tells you the magnitude; ten concrete rows tell you whether the predicate selects what you *believe* it selects. Most data incidents are a `WHERE` clause that meant something else.
* **A sanity ceiling.** If you expect ~10k rows and the script computes 4 million, that's a bug in the predicate, not a bigger job.

And the part everybody skips: a dry run predicts the change, but only a restore path lets you undo it. Confirm a snapshot covers the affected tables — and know the restore *time*; a backup you can't restore within tolerance isn't a safety net.

For targeted operations: archive the rows first.

```sql
CREATE TABLE _archive_2026_06_cart_purge AS
SELECT * FROM carts
WHERE status = 'abandoned' AND updated_at < :cutoff;
```

That one table makes the operation reversible in minutes instead of via a full restore. Drop it after an agreed retention period.

Protip: save the dry-run output (count + sample) in the PR or ticket *before* executing. Write the expectation down first.

## Don't ALTER columns in place

**Never change a column's type in place.** Add a new column, migrate onto it, and once everything has stabilized — every instance writing and reading the new one — drop the old. The safe operations are the additive ones: `ADD COLUMN` (nullable) is cheap, and `DROP COLUMN` is destructive but safe *once nothing reads the column*, which is exactly why it ships last and on its own. The moves in between — a `MODIFY` or `RENAME` on the column your whole fleet and every replica are reading right now — look like one-liners, but their blast radius is the entire table and every running instance at once.

### Change columns additively, not in place

The "safe rename" is a sequence, not a statement — and the same sequence covers any column change (retype, widen, split, merge), not just renames:

```text
   (renaming customer_code → customer_ref)
1. migration:  ADD COLUMN customer_ref (nullable)
2. code:       write to BOTH columns
3. script:     backfill customer_ref from customer_code   (batched, idempotent)
4. code:       read from customer_ref, stop writing customer_code
5. migration:  DROP COLUMN customer_code   (days later, after metrics
                                            confirm nothing reads it)
```

A direct `RENAME COLUMN` is atomic for the database but not for your fleet: during a rolling deploy, old instances are still querying the old name the instant the rename lands, and they start erroring. Routing through a new column keeps both names valid until every instance has moved over.

The principle under the whole sequence: for the entire migration period the schema lives in two shapes at once, so your application code has to work against **both** — the old column and the new — until every instance has rolled over. That compatibility window is exactly what steps 2–4 buy you; skip it and the change only survives if the database and the whole fleet flip in the same instant, which during a rolling deploy they never do.

Narrowing a type is the sharpest reason to work this way: a direct `ALTER` to a smaller width **silently truncates** every overflowing value under a non-strict MySQL — data loss with a green checkmark. Migrate through a new column and the overflow surfaces in the backfill, where *you* decide what happens to it instead of the engine deciding for you.

### What an in-place ALTER quietly takes from you

The additive rule isn't dogma — it's because of how much an in-place change silently destroys. Many engines treat a column modification as a *replacement*, not a patch:

```sql
-- existing: status VARCHAR(20) NOT NULL DEFAULT 'pending'

-- BAD: "just widen it" — MySQL silently drops NOT NULL and the DEFAULT
ALTER TABLE orders MODIFY COLUMN status VARCHAR(40);

-- to survive it, you must restate every attribute, by hand, every time
ALTER TABLE orders MODIFY COLUMN status VARCHAR(40) NOT NULL DEFAULT 'pending';
```

And that's the easy trap. For the in-place changes you genuinely can't avoid — the tightenings below — the same rule holds: **spell out the full definition and read the generated SQL before it ships.** Miss one attribute and it's silent data loss; the additive path never asks you to remember.

PostgreSQL's `ALTER COLUMN ... TYPE` preserves nullability and defaults, but ORM "change column" helpers often regenerate the full definition from whatever you declared — same trap, different layer.

For string columns, "complete" includes character set and collation: MySQL resets omitted ones to the table defaults, and a collation change alters which strings compare equal — silently breaking unique indexes and lookups.

Datetime type changes deserve their own paragraph of fear. Converting `DATETIME` ↔ `TIMESTAMP` in MySQL, or `timestamp` → `timestamptz` in PostgreSQL, rewrites every stored value through the session time zone unless you pin it. A uniform N-hour shift looks plausible row by row and errors nowhere.

And on a large table the `ALTER` itself can be the outage: the same statement is instant on one engine version and an hour-long lock on another. Know the locking behavior of every `ALTER` you write, reach for online schema-change tooling (gh-ost, pt-online-schema-change) on hot tables, and rehearse against production-sized data — a migration's profile on a 10k-row staging table tells you nothing about 200M rows.

### If you must tighten, gate it first

Tightening is the in-place change you can't always route around: `NOT NULL` and unique constraints land *after* the backfill, and each gets a pre-flight violation query before it ships.

Unique constraints: duplicates fail the `ALTER` at deploy time — but NULLs slip right past it. In MySQL and (by default) PostgreSQL a unique index permits *any number* of NULL rows, and the usual `GROUP BY ... HAVING COUNT(*) > 1` dup-check is no help here: it collapses every NULL into a single group and *reports* it as a false duplicate. Verify zero remaining NULLs separately, then pair the index with `NOT NULL`.

### Name your indexes and constraints

Index naming isn't an `ALTER` problem, but it's the same lesson — don't let the tool decide for you:

```sql
CREATE INDEX orders_customer_created_idx ON orders (customer_id, created_at);
```

Tool-generated names drift between environments (breaking the "drop the index" migration later), hit identifier length limits, and make incident work painful. A convention like `{table}_{columns}_{type}` costs nothing and pays forever.

## Don't use referential integrity

Heresy first, justification after: **keep foreign keys out of your database.** Declare the relationship in your code, index the column, and enforce integrity where you can actually reason about it — not in a constraint that fires on every write and fights you on every migration.

This is the one rule here people will argue about, so let me make the case.

### Why foreign keys cost more than they're worth

* **They lock the parent on every child write.** InnoDB takes a shared lock on the referenced row whenever you insert or update a child, so a hot parent — the `users` row of your busiest account, a popular `product` — turns into a contention point and a deadlock factory, all to enforce a check that says nothing about *your* business rules.
* **`ON DELETE CASCADE` is the exact thing the rest of this post warns against.** It's an unbounded, unbatched, un-throttled, un-dry-runnable `DELETE` fired as a side effect of another `DELETE`. A single `DELETE FROM users WHERE id = 42` can drag a million rows across ten tables down with it, holding locks the whole way — and you find out from the graphs. Everything in "Default to dry-run" becomes impossible the moment the database does the deleting for you.
* **They break online schema changes.** gh-ost refuses to touch a table that has foreign keys; pt-online-schema-change supports them only through fragile, risky workarounds. The tooling you most need on large tables (previous section) is exactly what FKs disable.
* **They don't survive scale or service boundaries.** You can't enforce a foreign key across shards, across a partition boundary, or across two services that each own their own database. The day `orders` and `customers` live apart, the guarantee you leaned on is gone — better to have never depended on it.
* **They turn routine data work into a fight.** Backfills, archival, soft deletes, reparenting, importing legacy rows in the "wrong" order — all of it has to be choreographed around a constraint that rejects every intermediate state, even a temporary and intentional one.

What FKs actually buy you — "a child never points at a missing parent" — is real, but it's a *guarantee about a single moment*, paid for on every write forever. You can get the same correctness far more cheaply.

### Enforce integrity where you can reason about it

Validate the relationship in the application, on write, alongside every other business rule — because the useful conditions live there anyway: the parent must exist *and* belong to the right tenant, be in the right state, not be archived. A foreign key can't express any of that.

**Index the referencing column yourself.** A FK in PostgreSQL does *not* auto-create an index on the child column (MySQL/InnoDB does); either way, add it explicitly — you want it for joins and lookups whether or not a constraint is attached.

```sql
CREATE INDEX orders_customer_id_idx ON orders (customer_id);
```

A bug can still slip an orphan through. Fine — you detect it, instead of locking your tables on every write to prevent it. The audit is a cheap anti-join you run on a schedule:

```sql
-- orphaned orders: child points at a parent that isn't there
SELECT o.id
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id
WHERE o.customer_id IS NOT NULL
  AND c.id IS NULL;
```

Alert on a non-zero count and you learn about drift in minutes — with the offending row IDs in hand, not a constraint violation blocking an unrelated deploy at 3 a.m.

### Handle deletes on purpose

Without `CASCADE`, deleting a parent becomes an explicit, reviewable operation — which is the entire point. Pick the policy per relationship:

* **Cascade in application code**, child tables first, using the batched, dry-run script from earlier. Now the cascade is countable, throttled, resumable and reversible.
* **Nullify** the reference (`SET customer_id = NULL`) when the child outlives the parent — an anonymized order, a retained event log.
* **Block** the delete with a pre-flight check ("does this customer still have orders?") when children should keep the parent alive — the same intent as `RESTRICT`, but with a real error message and your own timing.
* **Soft-delete** the parent and let a background job reconcile children, when an inline hard delete is too expensive to do on the request path.

Every one of these you can dry-run, batch, log and resume. A foreign key gives you exactly one option — `CASCADE` — and it's the one that violates every other rule in this post.

## Wrapping up

Before shipping anything that touches schema or data:

* migrations: DDL only
* data scripts: idempotent, batched, re-run tested
* dry-run by default, `--execute` explicit
* sanity ceiling on affected rows
* restore path before any destructive run
* change columns additively (add → migrate → drop), never `ALTER`/`RENAME` in place
* if you must `ALTER`, restate the full definition and name your indexes
* tighten only after a verified backfill
* no database-level foreign keys; enforce in code, index the column, audit for orphans
* destructive steps ship separately
* know your lock profile; rehearse on production-sized data

Schema is code — it can be reverted. **Data is forever.** Respect that, and you'll never have to explain why the orders table has 4 million rows with status `'pending'`.

COMMIT;
