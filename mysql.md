# MySQL

- [MySQL](#mysql)
  - [Privileges](#privileges)
  - [Control flow](#control-flow)
  - [General built-in functions](#general-built-in-functions)
    - [String functions](#string-functions)
      - [Character conversions](#character-conversions)
    - [Date functions](#date-functions)
    - [Numeric functions](#numeric-functions)
    - [Other/generic functions](#othergeneric-functions)
  - [Regular expressions (regexes)](#regular-expressions-regexes)
    - [Strategies](#strategies)
  - [XPath](#xpath)
  - [CSV Import/Export](#csv-importexport)
  - [Window functions](#window-functions)
    - [Define each window extent (`PARTITION BY`), and picking a row per window](#define-each-window-extent-partition-by-and-picking-a-row-per-window)
    - [Window function aggregates](#window-function-aggregates)
    - [Applications/examples](#applicationsexamples)
      - [Replace spaced coordinates with monotonically increasing values](#replace-spaced-coordinates-with-monotonically-increasing-values)
  - [Stored procedures](#stored-procedures)
    - [Cursors (with convenient example of looping)](#cursors-with-convenient-example-of-looping)
  - [Performance](#performance)
  - [Administration](#administration)
    - [Observe ALTER TABLE progress](#observe-alter-table-progress)
  - [Metadata](#metadata)

## Privileges

```sql
SHOW GRANTS for <user>[@'<host>']\G

-- *************************** 1. row ***************************
-- Grants for user@%: GRANT SELECT, .. ON *.* TO `user`@`1.2.3.4` WITH GRANT -- OPTION
-- *************************** 2. row ***************************
-- Grants for user@%: GRANT APPLICATION_PASSWORD_ADMIN, ... ON *.* TO `user`@`1.2.3.4` WITH GRANT OPTION
```

## Control flow

```sql
IF(condition, trueBranch, falseBranch)

CASE case_value
WHEN when_value THEN statement_list
[WHEN when_value THEN statement_list] ...
[ELSE statement_list]
END CASE
```

## General built-in functions

### String functions

```sql
TRIM(<expr>)                                       # Strips only spaces by default!
TRIM([BOTH|LEADING|TRAILING] str FROM <expr>)      # Strip characters; usable for newlines

SUBSTR(`field` (FROM|,) @start (FOR|,) @len)       # @start = 1-based; if start < 0, start from the end

INSTR(@str, @pattern)                              # 1-based; 0 if not found

CONCAT_WS(@separator, @fields..)                   # concatenate using the separator; ignores nulls
(L|U)CASE(@sstr)                                   # down/upcase
```

#### Character conversions

Convert from code point to character and viceversa:

```sql
SELECT CHAR(0xF09F91B8 USING utf8mb4);

-- +--------------------------------+
-- | CHAR(0xF09F91B8 USING utf8mb4) |
-- +--------------------------------+
-- | 👸                             |
-- +--------------------------------+

SELECT HEX(ORD('👸'));

-- +---------------+
-- | HEX(ORD('?')) |
-- +---------------+
-- | F09F91B8      |
-- +---------------+
```

### Date functions

```sql
NOW(), CURDATE(), CURTIME()                        # current time functions

# Read the next command note!
#
DATE_(ADD|SUB(@date, INTERVAL @n (DAY|SECOND|...))

# When adding a time to a time, use the following. DON'T use "+ INTERVAL xx MINUTES"!
#
ADDTIME(@time, 'hh:mm:ss');
```

### Numeric functions

```sql
CONV(@str, @from_base, @to_base)     # useful e.g. for hex conversion
TRUNCATE(@val, @places)              # truncate to decimal places (places can be negative), !!! always towards zero !!!
RAND()                               # random number; has low entropy
GET_BYTES(@number)                   # random number; has higher entropy. can use as `HEX(GET_BYTES)` in order to get a random hex string.
```

### Other/generic functions

```sql
IFNULL(<expr>, @substitute)
UUID()                               # generates a uuid
SLEEP(@secs)
LAST_INSERT_ID()                     # last inserted AUTO_INCREMENTed id
ROW_COUNT()                          # number of rows affected by last operation
GREATEST|LEAST(@values...)           # min/max on multiple values
```

## Regular expressions (regexes)

Metacharacters/functionalities supported:

- `^ $ . * + ? | {} [] \b`;
- `()` but not with groups capturing;
- `[:digit:]` and so on;
- backreferences is not supported;
- lookahead is not supported.

!! The backslash needs to be doubled where it's literal, e.g. `\\.`/`\\b` !!

`LIKE`-style operator:

```sql
-- Supports `match_type` as second argument (see `REGEXP_SUBSTR`).
--
SELECT 'abcd' RLIKE 'b.' `match`;
-- +-------+
-- | match |
-- +-------+
-- |     1 |
-- +-------+
```

Matching a substring:

```sql
-- - `position` (optional): 1-based, default=1;
-- - `occurrence` (optional): 1-based, default=1;
-- - `match_type` (optional): `c`ase sensitive, `m`ulti-line match, `n`ewlines matched by dot;
--
SET @position = 1, @occurrence = 2, @match_type = NULL;

SELECT REGEXP_SUBSTR('ab0b1', 'b[[:digit:]]', @position, @occurrence, @match_type) `match`;
-- +-------+
-- | match |
-- +-------+
-- |    b1 |
-- +-------+

SELECT
  REGEXP_SUBSTR('a\nb', 'a$', 1, 1, 'm') `match_m`,
  REPLACE(
    REGEXP_SUBSTR('a\nb', 'a.b', 1, 1, 'n'),
    '\n', '<\\n>'
  ) `match_n`
;
+---------+---------+
| match_m | match_n |
+---------+---------+
| a       | a<\n>b  |
+---------+---------+
```

Search/replace:

```sql
-- Supports the same options as `REGEXP_SUBSTR`.
-- !!! Don't forget to set <occurrence> to 0 for gsub !!!
--
SELECT REGEXP_REPLACE('a b b c', 'b', 'X', 1, 0) `replace`;
-- +---------+
-- | replace |
-- +---------+
-- | a X X c |
-- +---------+
```

### Strategies

Since capturing groups are not supported, using `REGEXP_REPLACE` can be the simplest option in some cases.  
Note that MySQL supports XPath, so input data like this is best handle throught that.

```sql
SET @input := '
<id>my_id</id>
<op>my_op</op>
';

SELECT
  REGEXP_REPLACE(
    REGEXP_SUBSTR(@input, 'my_id</id>.*?</', 1, 1, 'mn'),
    '^.*>|</$', '', 1, 0, 'mn'
  ) `operator`
;
-- +----------+
-- | operator |
-- +----------+
-- | my_op    |
-- +----------+
```

## XPath

Examples:

```sql
SET @input := '
<bookstore>
  <book>
    <title lang="en">Harry Potter</title>
    <price>29.99</price>
  </book>
  <book>
    <title lang="en">Learning XML</title>
    <price>39.95</price>
  </book>
</bookstore>
';

SELECT ExtractValue(@input, "//title[@lang='en']") `match`;
-- +---------------------------+
-- | match                     |
-- +---------------------------+
-- | Harry Potter Learning XML |
-- +---------------------------+

SELECT ExtractValue(@input, "/bookstore/book[price>35.00]/title") `match`;
-- +--------------+
-- | match        |
-- +--------------+
-- | Learning XML |
-- +--------------+
```

## CSV Import/Export

Field options are not defaults, so they must be set when needed.

If an import fails due to mysterious invalid characters, investigate the line terminators and characters via `hexdump` and `file -i`.

`LOAD DATA` syntax:

```sql
LOAD DATA [LOCAL] INFILE '/tmp/customers.csv'
[REPLACE|IGNORE]
INTO TABLE customers
[CHARACTER SET <charset>]
[FIELDS [TERMINATED BY ','] [ENCLOSED BY '"']]
[LINES TERMINATED BY '\n']
[IGNORE 1 LINES]
[(<column>, @variable, ...)]      # see example
[SET <column> = FUNC(@variable)]  # see example
;
```

Examples:

```sql
SELECT columns INTO OUTFILE 'outFile'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM fromPart;

# CSV contains only three fields; insert them in the specified colums. set the tenant_id field to 89, and set customer_id to NULL if it's blank.
#
LOAD DATA INFILE '/tmp/layouts_89.csv'
INTO TABLE events
FIELDS TERMINATED BY ','
( id, layout_id, @customer_id )
SET tenant_id = 89, customer_id = NULLIF(@customer_id, '');
```

## Window functions

Basic window functions example: compute the difference of a week of sales compared to the previous year.

`LAG(_expression_{, _N_})` is the value of _expression_ applied to the row preceding `N` positions (`N` defaults to 1).

It's crucial to keep in mind the [window functions processing order](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html):

> Query result rows are determined from the FROM clause, after WHERE, GROUP BY, and HAVING processing, and windowing execution occurs before ORDER BY, LIMIT, and SELECT DISTINCT.

as a consequence, it's not possible to filter out via `HAVING` (with this given structure) rows whose expression returns `NULL`.

In this (simplistic) example, can't use `YEAR()`/`WEEK()` separately, due to the week 0 being part of the previous year's last week.

```sql
SELECT
  YEARWEEK(created_at) `sales_year_week`,
  MIN(DATE(created_at)) `sales_week_start`,
  SUM(quantity) `tot_quantity`,
  SUM(quantity) - LAG(SUM(quantity), 52) OVER (ORDER BY created_at) `diff_quantity_52_weeks`
FROM sales
WHERE DATE(created_at) BETWEEN CURDATE() - INTERVAL (DAYOFWEEK(CURDATE()) - 1 ) DAY - INTERVAL 1 WEEK - INTERVAL 52 WEEK - INTERVAL 10 WEEK
                           AND CURDATE() - INTERVAL (DAYOFWEEK(CURDATE()) - 1 ) DAY - INTERVAL 1 WEEK
GROUP BY sales_year_week
ORDER BY sales_year_week;
```

When there are multiple equal windows, use named windows, otherwise, each window is evaluated separately:

```sql
# In this example, the window is the entire rowset, as we only define the ordering.
#
SELECT
  id,
  LAG(id) OVER w                       `reference_sale_id`,
  updated_at,
  LAG(updated_at) OVER w               `reference_sale_updated_at`,
  updated_at >= LAG(updated_at) OVER w `in_order`
FROM sales
WHERE special_customer
WINDOW w AS (ORDER BY id DESC)
```

### Define each window extent (`PARTITION BY`), and picking a row per window

`PARTITION BY` defines each window extent; in this case, the ordering must happen for each specific customer (id).

This example is a typical application - selecting a row within a window:

```sql
WITH ordered_items AS
(
  SELECT id, customer_id, active, expires_on,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY active DESC, id DESC) `item_order`
  FROM items
  WHERE customer_id IN (1, 2, 3)
)
SELECT * FROM ordered_items WHERE item_order = 1;
```

### Window function aggregates

```sql
ROW_NUMBER()                         # 1-based
LAG(@expr[, @position[, @default]])  # value of @expr from the previous row of the frame
                                     # @position defaults to 1; @default defaults to NULL, and can be an expression (great to avoid the first NULL)
FIRST_VALUE(@expr)                   # value of @expr from the first row of the frame
```

### Applications/examples

#### Replace spaced coordinates with monotonically increasing values

```sql
WITH
y_ords AS (
  SELECT
    y, ROW_NUMBER() OVER (ORDER BY y) - 1 `y_ord`
  FROM {table}
  WHERE {parent_fk} = ?
  GROUP BY y
),
x_ords AS (
  SELECT
    x, ROW_NUMBER() OVER (ORDER BY x) - 1 `x_ord`
  FROM {table}
  WHERE {parent_fk} = ?
  GROUP BY x
)
UPDATE {table} t
        JOIN y_ords yo USING (y)
        JOIN x_ords xo USING (x)
SET t.y = yo.y_ord, t.x = xo.x_ord
WHERE {parent_fk} = ?
```

## Stored procedures

### Cursors (with convenient example of looping)

```sql
# Script for iterating a block of queries through all the tenants.
#
# Each SELECT (in the SP) will output a result in tabular format (except when no rows are found).
# If the client is run in very silent mode (-ss) only the result values will be shown, however,
# by using column naming, a clear output can be achieved.
#
# ROW_COUNT() can be useful for logging.
#
# The SUBSTR is a workaround to avoid printing a confusing `0`.

SET @sleep_time = 5;

###################################################
# Stored procedure definition
###################################################

DROP PROCEDURE IF EXISTS ALL_TENANTS_OPERATION;

DELIMITER $$

CREATE PROCEDURE ALL_TENANTS_OPERATION()
MODIFIES SQL DATA
BEGIN
  DECLARE v_tenant_id   INT;
  DECLARE v_cur_notfound BOOL;

  # Cursors are closed automatically at the END of a SP, so there's no need to close them explicitly.
  #
  DECLARE cur_tenants CURSOR FOR SELECT id FROM tenants;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_cur_notfound := TRUE;

  OPEN cur_tenants;
  tenants_loop: LOOP
    # The number of variables specified must match the number of columns selected by the cursor.
    # For complex queries, where one wants to ignore selected fields, declared variables must be used;
    # a workaround is to declare a phony variable with `VARCHAR(255)` type, and specify it for all the
    # corresponding fields to ignore.
    #
    FETCH cur_tenants INTO v_tenant_id;
    IF v_cur_notfound THEN
      LEAVE tenants_loop;
    END IF;

    SELECT CONCAT(v_tenant_id, SUBSTR(SLEEP(@sleep_time), 0)) `Executing for tenant`,
           COUNT(*) `Accounts remaining`
    FROM tenants WHERE id > v_tenant_id;

    -- Insert the operation here, using v_tenant_id.
    --
    UPDATE ...;

    SELECT ROW_COUNT() `Updated rows`;
  END LOOP tenants_loop;
END$$

DELIMITER ;

###################################################
# Execution and procedure drop
###################################################

CALL ALL_TENANTS_OPERATION(); DROP PROCEDURE ALL_TENANTS_OPERATION;
```

## Performance

```sql
# Join order
# Reference: https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-join-order.

SELECT /*+ JOIN_ORDER(t2, t1) */ COUNT(*)
FROM table_1 t1 JOIN table_2 t2

# Disable optimizer switches:
#
SET GLOBAL optimizer_switch = "duplicateweedout=off";

# Display query trace:
#
SET optimizer_trace = "enabled=on";
SELECT * FROM information_schema.optimizer_trace\G
SET optimizer_trace = "enabled=off";

# Per-table (persistent) stats options.
# Reference: https://dev.mysql.com/doc/refman/8.0/en/innodb-persistent-stats.html.
#
ALTER TABLE seat_assignments STATS_SAMPLE_PAGES=50, STATS_AUTO_RECALC=0;
```

## Administration

### Observe ALTER TABLE progress

Reference: https://dev.mysql.com/doc/refman/8.0/en/monitor-alter-table-performance-schema.html

Example of ALTER TABLE monitoring, at different stages; test estimation is updated (increased) between stages:

```sql
SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM performance_schema.events_stages_current;

-- +------------------------------------------------------+----------------+----------------+
-- | EVENT_NAME                                           | WORK_COMPLETED | WORK_ESTIMATED |
-- +------------------------------------------------------+----------------+----------------+
-- | stage/innodb/alter table (read PK and internal sort) |         155094 |        7904880 |
-- +------------------------------------------------------+----------------+----------------+

-- +----------------------------------+----------------+----------------+
-- | EVENT_NAME                       | WORK_COMPLETED | WORK_ESTIMATED |
-- +----------------------------------+----------------+----------------+
-- | stage/innodb/alter table (flush) |        7054880 |        7901111 |
-- +----------------------------------+----------------+----------------+
```

The `alter table (flush)` stage empirically took from 10% to 100% of the total time before that stage.

Only the last stage `log apply table` causes contention; on the longest occurrence, it took ≈2.4% of the total time (44/1858").

## Metadata

Space occupation of InnoDB indexes (data):

```sql
# Remember that the table data is the primary index!
#
SELECT index_name, stat_value*@@innodb_page_size `size`
FROM mysql.innodb_index_stats
WHERE stat_name = 'size'
     AND (database_name, table_name) = ('my_db', 'my_table')
ORDER BY index_name;
```

Find the foreign keys for a table/column:

```sql
SELECT TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME, REFERENCED_TABLE_NAME, REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE (REFERENCED_TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME) = ('my_db', 'my_table', 'my_column');
```
