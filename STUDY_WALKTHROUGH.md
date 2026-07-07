
---

## Evening 1 — Schema + file 01 (foundations)

**Read:** `sql/00_schema.sql`, `sql/01_data_quality_checks.sql`

Concepts to be able to explain out loud:
- **Primary key / foreign key:** `packages.route_id` points to `routes.route_id`.
  Say it as: "every package knows which route carried it."
- **LEFT JOIN + IS NULL** = "find rows with no match." Query 1.2 keeps ALL
  packages and looks for ones whose route doesn't exist.
- **GROUP BY ... HAVING:** WHERE filters rows before grouping, HAVING filters
  groups after. Query 1.5 uses HAVING COUNT(*) > 1 to catch duplicates.

**Do:** run each query. Then break one on purpose — change LEFT JOIN to INNER
JOIN in 1.2 and observe why it can no longer find orphans. (INNER JOIN drops
the very rows you're hunting.)

**Interview line:** "I always start with data-quality checks — orphan records,
null consistency, duplicates — because a KPI on broken data is worse than no KPI."

## Evening 2 — File 02 (KPIs) + file 04 (failure analysis)

Concepts:
- **The percentage trick:** `AVG(CASE WHEN status='delivered' THEN 1.0 ELSE 0 END)`
  turns a yes/no column into a rate. This appears everywhere in the project —
  master it and half the repo is yours.
- **CTE (`WITH x AS (...)`):** a named sub-result. Read 2.2 top-to-bottom:
  first build `final_delivery` (one row per delivered package), then join it.
- **Pivot with CASE (4.2):** one SUM(CASE...) column per failure reason turns
  rows into columns for side-by-side comparison.
- **`strftime`:** SQLite's date formatter. `%Y-%W` = year-week, `%w` = weekday.

**Do:** write from scratch (no peeking): "failure rate per service_type
(standard vs prime_same_day)." It's the 2.1 pattern with one extra JOIN + GROUP BY.

## Evening 3 — File 03 (grain) + file 05 (window functions)

Concepts:
- **Grain (3.1)** — the most senior-sounding thing in the repo. Routes and
  packages exist at different levels of detail. If you join them raw and then
  AVG a route column, every route row is repeated once per package and the
  average is wrong. Fix: aggregate each grain in its own CTE first, join after.
- **julianday(a) - julianday(b)** = days between two dates (5.1 computes tenure
  *at the time of the route*, not today — be ready to explain why that matters).
- **RANK() OVER (PARTITION BY station ORDER BY x)** = rank within each station
  without collapsing rows. Window functions read the table "sideways."

**Do:** modify 5.2 to use ROW_NUMBER() instead of RANK() and explain the
difference (ties).

## Evening 4 — File 06 (hardest) + mock interview

- **LAG(pkgs) OVER (PARTITION BY station ORDER BY week)** = "previous week's
  value on the same row" → week-over-week growth without self-joins.
- **The runway projection (6.2):** current volume grows at g per week; weeks
  until it hits capacity solves capacity = current × (1+g)^w, i.e.
  w = ln(capacity/current) / ln(1+g). You wrote compound growth in SQL.

**Mock interview — answer these out loud:**
1. Walk me through how you'd find the worst-performing station. (File 03 story)
2. What's the difference between WHERE and HAVING?
3. Why did you compute tenure against route_date instead of today?
4. What's a window function? Give an example from your project.
5. What would you do differently with real data? (README section 6)

---

## Honest framing for interviews

Say it exactly like it happened: *"I built a synthetic last-mile dataset and a
SQL analysis on top of it to teach myself SQL — I chose last-mile because I know
that domain from working in Amazon Logistics operations. I used AI assistance to
scaffold the project and then worked through every query until I could write the
patterns myself."* Nobody penalizes AI-assisted learning in 2026; they penalize
people who can't explain their own repo.
