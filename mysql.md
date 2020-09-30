# MySQL

- [MySQL](#mysql)
  - [Privileges](#privileges)
  - [Built-in functions](#built-in-functions)
    - [String functions](#string-functions)
      - [Character conversions](#character-conversions)
    - [Regular expressions (regexes)](#regular-expressions-regexes)
      - [Strategies](#strategies)
    - [XPath](#xpath)
  - [Window functions](#window-functions)
    - [Define each window extent (`PARTITION BY`), and picking a row per window](#define-each-window-extent-partition-by-and-picking-a-row-per-window)
    - [Window function aggregates](#window-function-aggregates)

## Privileges

```sql
SHOW GRANTS for <user>[@'<host>']\G

-- *************************** 1. row ***************************
-- Grants for user@%: GRANT SELECT, .. ON *.* TO `user`@`1.2.3.4` WITH GRANT -- OPTION
-- *************************** 2. row ***************************
-- Grants for user@%: GRANT APPLICATION_PASSWORD_ADMIN, ... ON *.* TO `user`@`1.2.3.4` WITH GRANT OPTION

```

## Built-in functions

### String functions

```sql
TRIM(<expr>)                                             # Strips only spaces by default!
TRIM([BOTH|LEADING|TRAILING] <str> FROM <expr>)          # Strip characters; usable for newlines
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

### Regular expressions (regexes)

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

#### Strategies

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

### XPath

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
