# MySQL

- [MySQL](#mysql)
  - [Privileges](#privileges)
  - [Control flow](#control-flow)
  - [General statements](#general-statements)
  - [General built-in functions](#general-built-in-functions)
    - [String functions](#string-functions)
      - [Character conversions](#character-conversions)
    - [Date functions](#date-functions)
    - [Numeric functions](#numeric-functions)
    - [Aggregates](#aggregates)
    - [Other/generic functions](#othergeneric-functions)
  - [Regular expressions (regexes)](#regular-expressions-regexes)
    - [Strategies](#strategies)
  - [JSON + MVI and array storage](#json--mvi-and-array-storage)
  - [XPath](#xpath)
  - [CSV Import/Export](#csv-importexport)
  - [Dates](#dates)
    - [General useful notes](#general-useful-notes)
    - [Formatting/parsing](#formattingparsing)
  - [Window functions](#window-functions)
    - [Define each window extent (`PARTITION BY`), and picking a row per window](#define-each-window-extent-partition-by-and-picking-a-row-per-window)
    - [Window function aggregates](#window-function-aggregates)
    - [Applications/examples](#applicationsexamples)
      - [Replace spaced coordinates with monotonically increasing values](#replace-spaced-coordinates-with-monotonically-increasing-values)
  - [Stored procedures](#stored-procedures)
    - [Variables](#variables)
    - [Control flow syntax](#control-flow-syntax)
    - [Statements output and SLEEP](#statements-output-and-sleep)
    - [Exceptions handling](#exceptions-handling)
    - [Cursors (with example procedure)](#cursors-with-example-procedure)
  - [Performance](#performance)
    - [Profiling](#profiling)
    - [Dynamic SQL](#dynamic-sql)
  - [Administration](#administration)
    - [Non-blocking schema changes](#non-blocking-schema-changes)
    - [Observe ALTER TABLE progress](#observe-alter-table-progress)
  - [Client/server](#clientserver)
    - [Find configuration files used](#find-configuration-files-used)
  - [Metadata](#metadata)
  - [Convenient operations](#convenient-operations)
    - [Skip the indexes on mysqldump dumps](#skip-the-indexes-on-mysqldump-dumps)

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

## General statements

```sql
# Note that values outside the @values list will take priority over the ones inside, so for some cases
# it's necessary to use the DESC modifier.
#
ORDER BY FIELD( field, @values...)
```

## General built-in functions

### String functions

```sql
TRIM(<expr>)                                       # Strips only spaces by default! Defaults to `BOTH` sides
TRIM([BOTH|LEADING|TRAILING] str FROM <expr>)      # Strip characters; usable for newlines

SUBSTR(`field` (FROM|,) @start (FOR|,) @len)       # @start = 1-based; if start < 0, start from the end

INSTR(@str, @pattern)                              # 1-based; 0 if not found

CONCAT_WS(@separator, @fields..)                   # concatenate using the separator; ignores nulls
(L|U)CASE(@sstr)                                   # down/upcase
```

`SUBSTR` doesn't support end position (only length). If one wants to strip values around a string, can use `TRIM`:

```sql
SELECT TRIM('"' FROM '"aa"aa"');
-- aa"aa
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

### Aggregates

```sql
GROUP_CONCAT(@field[, @separator])   # ignores NULL fields, but returns NULL if there are no non-null values; @separator defaults to `,`
```

### Other/generic functions

```sql
IFNULL(<expr>, @substitute)
UUID()                               # generates a uuid
SLEEP(@secs)
LAST_INSERT_ID()                     # last inserted AUTO_INCREMENTed id
ROW_COUNT()                          # number of rows affected by last operation (NOT rows changed!)
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

## JSON + MVI and array storage

Add a JSON column and, separately, a MVI, allowing for arrays storage:

```sql
CREATE TABLE clients (
  id          INT PRIMARY KEY,
  client_tags JSON
)
SELECT 1 `id`, '["foo", "bar", "baz"]' `client_tags`;

-- Limited to 512!
ALTER TABLE clients ADD KEY client_tags ( (CAST(client_tags -> '$' AS CHAR(512) ARRAY)) );

EXPLAIN FORMAT=TREE SELECT * FROM clients WHERE 'foo' MEMBER OF (client_tags -> '$')\G
-- -> Filter: json'"foo"' member of (cast(json_extract(client_tags,_utf8mb4'$') as char(64) array))  (cost=0.35 rows=1)
--     -> Index lookup on clients using client_tags (cast(json_extract(client_tags,_utf8mb4'$') as char(64) array)=json'"foo"')  (cost=0.35 rows=1)
```

JSON search functions:

```sql
-- `one`: return first match; `All`: return all
-- the return type of `All` is inconsistent; it depends on the number of elements found.
--
SELECT JSON_SEARCH(client_tags, 'one', 'bar') FROM clients;
-- "$[1]"
SELECT JSON_SEARCH('["bar", "bar"]', 'All', 'bar');
-- ["$[0]", "$[1]"]
SELECT JSON_SEARCH(client_tags, 'All', 'bar') FROM clients;
-- "$[1]"
```

JSON modification functions. WATCH OUT! Operating on a null node (/column) will return NULL.

```sql
SELECT JSON_ARRAY_APPEND(client_tags, '$', 'qux') FROM clients;
-- ["foo", "bar", "baz", "qux"]

SELECT JSON_ARRAY_INSERT(client_tags, '$[1]', 'qux') FROM clients;
-- ["foo", "qux", "bar", "baz"]

SELECT JSON_REMOVE(client_tags, '$[1]') FROM clients;
-- ["foo", "baz"]

-- Removal by value can't be directly done; must search+remove (and trim)
SELECT JSON_REMOVE(client_tags, TRIM(BOTH '"' FROM JSON_SEARCH(client_tags, 'one', 'bar'))) FROM clients;
-- ["foo"]

-- Set/replace. WATCH OUT! See result below for out-of-bounds cases.
--
SELECT JSON_SET(client_tags, '$[5]', 'qux') FROM clients;
-- ["foo", "bar", "baz", "qux"]

-- Insert only; never replace. Same considerations above about indexing.
--
SELECT JSON_INSERT(client_tags, '$[5]', 'qux') FROM clients;
-- ["foo", "bar", "baz", "qux"]

-- Replace only; never insert if not existing.
--
SELECT JSON_REPLACE(client_tags, '$[5]', 'qux') FROM clients;
-- ["foo", "bar", "baz"]
```

Convert an array to a table:

```sql
-- Operation a single string
--
SELECT *
FROM
  JSON_TABLE(
    '[5, 6, 70]', -- JSON doc
    "$[*]"        -- JSON path
    COLUMNS(
      Value VARCHAR(16) PATH "$" -- Mapping; if the data type is CHAR, must specify the size, otherwise↵
                                 -- CHAR(1) is used, and data is truncated!
      ERROR ON ERROR
    )
  ) data;
-- +-------+
-- | Value |
-- +-------+
-- | 5     |
-- | 6     |
-- | 70    |
-- +-------+

-- Compute the per-tag count for a given tenant.
--
-- 2.6" - pretty good.
-- total sources: 8.0M; tenant: 630k (index on tenant_id)
--
SELECT t.tag, COUNT(*)
FROM
  source s,
  JSON_TABLE(s.tags, '$[*]' COLUMNS(tag VARCHAR(255) PATH '$' ERROR ON ERROR)) t
WHERE s.tenant_id = 0
GROUP BY t.tag;

-- Selecting the unique tags for a given tenant
-- A bit faster: 2.4"
--
SELECT DISTINCT t.tag
FROM
  source s,
  JSON_TABLE(s.tags, '$[*]' COLUMNS(tag VARCHAR(255) PATH '$' ERROR ON ERROR)) t
WHERE s.tenant_id = 0;

-- Counting the unique tags for a given tenant: 2.6".
-- It seems that there are no specific optimizations; there could be in the future.
--
EXPLAIN FORMAT=TREE
SELECT COUNT(DISTINCT t.tag)
FROM
  source s,
  JSON_TABLE(s.tags, '$[*]' COLUMNS(tag VARCHAR(255) PATH '$' ERROR ON ERROR)) t
WHERE s.tenant_id = 0\G
*************************** 1. row ***************************
-> Aggregate: count(distinct t.tag)
    -> Nested loop inner join
        -> Index lookup on s using tenant_id (tenant_id=0)  (cost=229585.10 rows=1157816)
        -> Materialize table function

-- For reference, a scan of the filtered records takes 0.6":
--
SELECT COUNT(*) FROM source where tenant_id = 0 AND unindexed_column = 'xxx';
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

## Dates

### General useful notes

```sql
# Current timezone:
#
SELECT TIMEDIFF( NOW(), CONVERT_TZ( NOW(), @@session.time_zone, '+00:00' ) );

# Don't use `-` to substract timestamps!! Use:
#
TIMESTAMPDIFF(SECOND, timestamp1, timestamp2);
```

### Formatting/parsing

```sql
DATE_FORMAT(@date, @format);
STR_TO_DATE(@date, @format);
```

- `%a` : Abbreviated weekday name (Sun..Sat)
- `%b` : Abbreviated month name (Jan..Dec)
- `%c` : Month, numeric (0..12)
- `%D` : Day of the month with English suffix (0th, 1st, 2nd, 3rd, …)
- `%d` : Day of the month, numeric (00..31)
- `%e` : Day of the month, numeric (0..31)
- `%f` : Microseconds (000000..999999)
- `%H` : Hour (00..23)
- `%h` : Hour (01..12)
- `%I` : Hour (01..12)
- `%i` : Minutes, numeric (00..59)
- `%j` : Day of year (001..366)
- `%k` : Hour (0..23)
- `%l` : Hour (1..12)
- `%M` : Month name (January..December)
- `%m` : Month, numeric (00..12)
- `%p` : AM or PM
- `%r` : Time, 12-hour (hh:mm:ss followed by AM or PM)
- `%S` : Seconds (00..59)
- `%s` : Seconds (00..59)
- `%T` : Time, 24-hour (hh:mm:ss)
- `%U` : Week (00..53), where Sunday is the first day of the week
- `%u` : Week (00..53), where Monday is the first day of the week
- `%V` : Week (01..53), where Sunday is the first day of the week; used with %X
- `%v` : Week (01..53), where Monday is the first day of the week; used with %x
- `%W` : Weekday name (Sunday..Saturday)
- `%w` : Day of the week (0=Sunday..6=Saturday)
- `%X` : Year for the week, where Sunday is the first day of the week, numeric, four digits; used with %V
- `%x` : Year for the week, where Monday is the first day of the week, numeric, four digits; used with %v
- `%Y` : Year, numeric, four digits
- `%y` : Year, numeric (two digits)
- `%%` : A literal "%" character

```sql
# 0427 = Sunday

# Week starts on Sunday
#
str_to_date(date_format('20080426', '%X%V 0'), '%X%V %w'); -- 04-20
str_to_date(date_format('20080427', '%X%V 0'), '%X%V %w'); -- 04-27
str_to_date(date_format('20080428', '%X%V 0'), '%X%V %w'); -- 04-27

# Week starts on Monday
#
str_to_date(date_format('20080426', '%x%v 1'), '%x%v %w'); -- 04-21
str_to_date(date_format('20080427', '%x%v 1'), '%x%v %w'); -- 04-21
str_to_date(date_format('20080428', '%x%v 1'), '%x%v %w'); -- 04-28
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

### Variables

```sql
-- The default is optional.
DECLARE deletion_limit INT DEFAULT 50 * 1000;
DECLARE max_table_id   INT DEFAULT (SELECT MAX(id) FROM mytable);

-- Declared variables can be used in some places where standard variables can't.
DELETE FROM mytable LIMIT deletion_limit;
```

### Control flow syntax

Control flow (there is no for loop!):

```sql
IF @condition THEN
  # block
ELSEIF @condition THEN
  # block
ELSE
  # block
END IF;

# Exit from a cycle.
LEAVE label;

WHILE condition DO
  # block
END WHILE;

REPEAT
  # block
UNTIL @condition
END REPEAT;

LOOP
  # block
END LOOP;
```

Common properties:

```sql
-- The label is optional if unused, and it can also be on a separate line from the cycle token, although
-- the manual doesn't show it.
--
label_name: STATEMENT
  # Repeat the cycle
  ITERATE label_name;
END STATEMENT label_name; -- The label name is optional.
```

### Statements output and SLEEP

Only SELECTs print an output, and only if they're *not* assigned (see SLEEP below).

Each SELECT (in the SP) will output a result in tabular format (except when no rows are found).
If the client is run in very silent mode (-ss) only the result values will be shown, however, by using column naming, a clear output can be achieved.

In order to sleep without output, just assign to a bogus variable:

```sql
SET v_null_output = (SELECT SLEEP(1.0));
```

### Exceptions handling

```sql
DECLARE EXIT HANDLER FOR SQLEXCEPTION
BEGIN
  ROLLBACK;
  SELECT 'An error has occurred, operation rollbacked and the stored procedure was terminated';
END;
```

### Cursors (with example procedure)

Script for iterating a block of queries through all the tenants:

```sql
###################################################
# Stored procedure definition
###################################################

DROP PROCEDURE IF EXISTS ALL_TENANTS_OPERATION;

DELIMITER $$

CREATE PROCEDURE ALL_TENANTS_OPERATION()
MODIFIES SQL DATA
BEGIN
  DECLARE v_sleep_time = 5;

  DECLARE v_tenant_id    INT;
  DECLARE v_cur_notfound BOOL;
  DECLARE v_null_output  CHAR;

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

    SELECT v_tenant_id `Executing for tenant`, COUNT(*) `Accounts remaining`
    FROM tenants WHERE id > v_tenant_id;

    SET v_null_output = (SELECT SLEEP(v_sleep_time));

    # Insert the operation here, using v_tenant_id.
    #
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
# Query hints: Join order
# Reference: https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-join-order.

SELECT /*+ JOIN_ORDER(t2, t1) */ COUNT(*)
FROM table_1 t1 JOIN table_2 t2

# Disable optimizer switches:
#
SET GLOBAL optimizer_switch = "duplicateweedout=off";

# Per-table (persistent) stats options.
# Reference: https://dev.mysql.com/doc/refman/8.0/en/innodb-persistent-stats.html.
#
ALTER TABLE seat_assignments STATS_SAMPLE_PAGES=50, STATS_AUTO_RECALC=0;

# On batch inserion, disable key checks:
#
SET unique_checks=0;
SET foreign_key_checks=0;
```

### Profiling

General profiler:

```sql
SET PROFILING=1;

# execute query

SHOW PROFILES;
SHOW PROFILE FOR QUERY @query_id;

SELECT
  STATE,
  SUM(DURATION)            `Total_R`,
  ROUND(
    100 * SUM(DURATION) / (
      SELECT SUM(DURATION)
      FROM INFORMATION_SCHEMA.PROFILING
      WHERE QUERY_ID = @query_id
    ) , 2) `Pct_R`,
  COUNT(*)                 `Calls`,
  SUM(DURATION) / COUNT(*) `R/Call`
FROM INFORMATION_SCHEMA.PROFILING
WHERE QUERY_ID = @query_id
GROUP BY STATE
ORDER BY Total_R DESC;
```

Query optimizer trace:

```sql
SET optimizer_trace = "enabled=on";
SELECT * FROM information_schema.optimizer_trace\G
SET optimizer_trace = "enabled=off";
```

Digest slow query log, using Maatkit:

```sh
mk-query-digest /path/to/*-slow.log
```

### Dynamic SQL

- `@SQL_STRING` must be either a literal, or a variable - functions are illegal;
- remember that ROW_COUNT() is valid only after EXECUTE.

```sql
PREPARE pst_count FROM @SQL_STRING;
EXECUTE pst_count;
DEALLOCATE PREPARE pst_count;
```

## Administration

### Non-blocking schema changes

Gh-ost (works also on tables with triggers).

This is the configuration for running it on the master (has more overhead than running from the slave); with `--execute`, changes are applied, and the old table is kept, without it, changes are not applied:

```sh
gh-ost \
  --user=$USER --password=$PWD --host=localhost \
  --database=ticketsolve_production --table=comments \
  --alter="ENGINE=InnoDB" \
  --exact-rowcount --verbose \
  --allow-on-master \
  --execute \
;
```

`pt-online-schema-change` doesn’t work on tables with triggers; crashed on production:

```sh
pt-online-schema-change --execute --alter "MODIFY section VARCHAR(40) NOT NULL DEFAULT ''" D=db_name,t=table_name,u=root,p=root_pwd
```

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

## Client/server

### Find configuration files used

```sh
ls -l $(mysqld --verbose --help | awk "/Default options/ { getline; gsub(\"~\", \"$HOME\", \$0); print }") 2> /dev/null
```

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

## Convenient operations

### Skip the indexes on mysqldump dumps

Replace keys with null operations; valid alternatives:

- on `ALTER TABLE`
  - `ALGORITHM=DEFAULT`
  - `LOCK=DEFAULT`
  - `ENABLE KEYS`
  - `ADD CHECK (TRUE)` : adds a check
- on `CREATE TABLE`
  - `CHECK (TRUE)` : adds a check

`ALTER TABLE` key additions is produced by the Percona distro with `--innodb-optimize-keys`.

We can't just remove the lines, because if they're at the end of the table def, they'll leave the previous statement with a terminating comma.  
The leading comma could be deleted with some text processing trickery, but this solution is simpler.

```sh
mysqldump $db | perl -pe 's/^  (UNIQUE |FULLTEXT )?KEY.+?(,)?$/  CHECK(TRUE)$2'
```
