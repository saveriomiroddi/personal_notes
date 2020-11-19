# PostgreSQL

- [PostgreSQL](#postgresql)
  - [General operations](#general-operations)

## General operations

Connect via client:

```sh
psql <database_name>
```

Create a database:

```sql
# Don't forget that `schema` and `database` are two different concepts in PGSQL!
CREATE DATABASE <db_name>
```

Describe a table:

```sql
# `\d+` additionally has: `Storage`,`Stats target`,`Description`
\d <table_name>
```

Dump/restore a database:

```sh
# `F`ormat: `p`=SQL
# other formats: [d]irectory, [t]ar -> require pg_restore
#
pg_dump -F p <db_name> > dump.sql
psql <db_name> < dump.sql
```

Explain a query (with the most information):

```sql
EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON)
SELECT carts2.*
FROM carts2
     JOIN payments2 ON carts2.id = payments2.cart_id
WHERE payments2.type = 'CashPayment'
;
```

Explain a query in the web: http://tatiyants.com/pev/#/plans/new

Explain a query via PGAdmin: open query tool -> click explain analyze (right of the hand icon)
