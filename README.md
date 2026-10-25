# SQL Query Optimization in MySQL
 
Hands-on study of how MySQL 8 executes queries and how small changes to a query (or its indexes) change the execution plan. Every experiment is measured with `EXPLAIN ANALYZE` on the classic **employees** sample database (~300K employees, ~2.8M salary rows).
 
📄 **Full report with query plans and screenshots:** [`SQL_Query_Optimization_Report.pdf`](SQL_Query_Optimization_Report.pdf)
 
## Topics
 
| # | Topic | What was compared | Key finding |
|---|-------|-------------------|-------------|
| 1 | Select only needed columns | `SELECT *` vs `SELECT emp_no, hire_date` | Same plan and cost (full table scan); the narrower query is cheaper to build, transfer and display, and can use a covering index. |
| 2 | Indexes | `first_name = 'Mary'` without vs with an index | Without an index MySQL scans all 300,024 rows to find 224. The indexed run requires a local database (see note below). |
| 3 | `UNION` vs `UNION ALL` | Dedup via temporary table vs plain append | Same 251 rows, but cost 76,590 vs 60,411. Use `UNION ALL` when duplicates are impossible. |
| 4 | Subquery vs `JOIN` | `IN (subquery)` vs `JOIN` | MySQL 8 rewrites `IN` into a semijoin, so both plans cost the same. A `JOIN` is the more predictable choice in harder cases. |
| 5 | Optimizing `JOIN`s | All salaries vs latest salary per employee | Returning ~9x fewer rows was **not** faster (2,080 ms vs 1,850 ms) because the plan is more complex. Filtering on `to_date` ran in 1,334 ms but is not equivalent. |
 
## Highlights
 
- Reading `EXPLAIN ANALYZE` output: access method, estimated cost vs actual time, rows scanned vs rows returned.
- Why estimated cost and wall-clock time can disagree (for example, `UNION` vs `UNION ALL`).
- Why fewer returned rows does not guarantee a faster query.
- Writing SARGable predicates: date ranges instead of `YEAR()`/`MONTH()` on a column.
## Limitations (honest notes)
 
- **Task 2, variant B was not measured.** The shared course server denied `CREATE INDEX` (`Error 1142`), so the indexed run is described from how InnoDB works, not from a measurement. It needs a local copy of the database.
- **Single runs.** Most timings come from one execution, so differences of a few milliseconds are within noise; compare plans and cost rather than raw milliseconds.
- **Window-function variant failed.** A `ROW_NUMBER() OVER (...)` version of task 5 hit `Error 1114: table is full` on the shared server because it materializes all 2.8M rows in a temporary table.
- **Task 5, variant C is not equivalent to B.** It returns only employees with a currently active salary (240,124 rows vs 300,024).
## Stack
 
MySQL 8  · DBeaver · `EXPLAIN ANALYZE`
 
## Repository layout
 
```
.
├── README.md
├── SQL_Query_Optimization_Report.pdf
   
```
 
## Dataset
 
[employees sample database](https://github.com/datacharmer/test_db) (MySQL). Tables used: `employees`, `dept_manager`, `salaries`.
 
