# plsqlmig: control-flow PySpark converter (`--pyspark-cf-out`)

This feature is additive. `--pyspark-out` and `--pyspark-all` produce exactly the same output as before.

```
python -m plsqlmig.cli sample/*.sql --dry-run --pyspark-cf-out out_cf.py                  # CONVERT/REWRITE only
python -m plsqlmig.cli sample/*.sql --dry-run --pyspark-all --pyspark-cf-out out_cf.py    # every routine
```

## What is translated

* `create_tables()`: CREATE TABLE / ALTER TABLE ADD / CREATE VIEW DDL found in the input files, mapped to Spark types (`NUMBER` → `DECIMAL(38,10)`, `VARCHAR2` → `VARCHAR`) as Delta tables with IF NOT EXISTS. Run it once before the procedures. Sequences, indexes, synonyms and DB links are listed as notes.
* `EXECUTE IMMEDIATE 'CREATE TABLE ...'` (including CTAS) inside procedures is transpiled the same way.

* Variables, constants, package globals, IN/OUT parameters (OUT values are returned).
* IF / ELSIF / ELSE, WHILE, basic LOOP, numeric FOR, EXIT/CONTINUE WHEN, RETURN.
* Cursor FOR loops:
  * Rewritten **set-based** (snapshot DataFrame → MERGE / INSERT…SELECT) when each row is independent.
  * Otherwise converted **row-by-row**: `for rec in _rows(...)`.
* OPEN/FETCH/CLOSE and `%NOTFOUND`, SELECT INTO (NO_DATA_FOUND / TOO_MANY_ROWS), BULK COLLECT, `SQL%ROWCOUNT`.
* EXCEPTION handlers (try/except), RAISE_APPLICATION_ERROR, nested procedures and calls between routines.
* PL/SQL functions used inside SQL are registered as Spark UDFs.
* Sequences, SYSDATE arithmetic, and NVL/DECODE/TO_CHAR shims.

## Must review before production

* UPDATE, DELETE and MERGE need Delta or Iceberg tables.
* COMMIT, ROLLBACK and AUTONOMOUS_TRANSACTION have no Spark equivalent (they are marked with a note).
* Lines marked `TODO` include external or unknown calls, CASE statements and database links.
* Correctness must be proven against Oracle output on real data.

## Test

`tests/run_tests.py` runs the generated code on local Spark 4.x with seeded tables (needs pyspark and sqlglot).
