# DECISIONS.md

1. **Indexes.**
   - `employees`: Unique index on `emp_code` enforces business identifier uniqueness and accelerates lookups. Compound indexes on `(department, joined_on)` and `(joined_on, department)` optimize department filtering and date-bounded headcount queries.
   - `attendance_logs`: Unique compound index on `(emp_code, date)` guarantees natural key uniqueness and prevents concurrent double punch-ins. `(date, emp_code)` supports paginated listing without in-memory sorting. `(date, late_minutes, emp_code)` backs the late leaderboard. `(emp_code, punch_in)` optimizes finding the latest open punch-in.
   - We rejected indexing `status` alone due to low cardinality, as date and employee prefix filters are far more selective.

2. **Punch-in race.**
   Both requests compute identical `(emp_code, date)` keys and attempt `insert_one`. MongoDB's unique compound index atomically accepts the first write and rejects the concurrent duplicate with a `DuplicateKeyError`. Our code catches this exception and returns HTTP 409 Conflict, ensuring exactly one 201 response.

3. **Ties.**
   Our pipeline applies MongoDB's `$setWindowFields` using `$rank` (standard competition ranking: 1, 2, 2, 4), followed by filtering `rank <= limit`. When employees tie at the cutoff rank, both satisfy `rank <= limit`, so both are returned, breaking ties by `emp_code` ascending.

4. **Headcount.**
   The aggregation pipeline starts from `employees` filtered by `joined_on <= end_of_month` and joins logs via `$lookup`. Headcount counts matching employee documents (`$sum: 1`). Employees with zero logs yield an empty logs array, preserving their headcount contribution while contributing zero to attendance totals.

5. **One thing you would change for 100x data.**
   At 100x scale (~10M documents), on-the-fly multi-stage aggregations would strain the primary cluster. We would introduce asynchronous daily/monthly rollup summary collections maintained incrementally via MongoDB Change Streams on attendance events.
