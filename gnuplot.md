# gnuplot

- [gnuplot](#gnuplot)
  - [Notes](#notes)
  - [General options](#general-options)
  - [Syntax/Basic use cases](#syntaxbasic-use-cases)
    - [Important syntax](#important-syntax)
    - [Single line, with dates](#single-line-with-dates)
    - [Multiple lines, with data `x, line₁y, line₂y, ...`](#multiple-lines-with-data-x-liney-liney-)
    - [Multiple lines](#multiple-lines)
      - [with dataset header and data `x, y`](#with-dataset-header-and-data-x-y)
      - [with data `lineN, x, y`, simple but less pretty](#with-data-linen-x-y-simple-but-less-pretty)
      - [with data `lineN, x, y`, prettier but more complex (requires Perl), with conveniences](#with-data-linen-x-y-prettier-but-more-complex-requires-perl-with-conveniences)
  - [Maths](#maths)
  - [Aesthetics](#aesthetics)
  - [More complex cases](#more-complex-cases)
    - [Two lines, with different scales](#two-lines-with-different-scales)
  - [Ready scripts](#ready-scripts)
    - [Plot base common global status values](#plot-base-common-global-status-values)
    - [Plot global status values, using deltas](#plot-global-status-values-using-deltas)
    - [Plot multiple global status values, in a single chart](#plot-multiple-global-status-values-in-a-single-chart)

## Notes

The `pl` language is used (Perl), which is not correct, but close enough.

## General options

Commandline:

```sh
# Batch mode: displays the diagram and exits. WATCH OUT! No effect if the output if a file.
#
gnuplot --persist $file

# Execute. !! the command must be semicolon-separated !!
#
gnuplot -e "commands..."
```

Language:

```pl
# Accept a CSV data file
#
set datafile separator ','

# Write output to image file; format available: `svg`,`png`, ...
# Enhanced has better text output.
#
# For svg, append `background rgb 'white'` for a white background (default is transparent).
#
set terminal $format enhanced
set output 'output.$format'

# Show the diagram, and wait for close before exit; requires return to be tapped - this can be worked around by
# `echo`ing a newline to stdin.
#
pause mouse close

# Debug (`-` signifies stdout)
#
set print '-'
print ...
```

## Syntax/Basic use cases

### Important syntax

A datafile can be divided in datasets, separated by two blank lines. Blank lines before the first dataset are ignored.

In order to read data from stdin, just use `-` as filename. WATCH OUT!! Doesn't work with for loop, since they read the input multiple times.

### Single line, with dates

```pl
# Input format:
#
#     06-12 10
#     06-13 11

set xdata time         # parse x as date
set timefmt "%m-%d"    # set date format

plot "data.txt" \
  using 1:2 \          # use columns (1-based): 1=x, 2=y
  with linespoints \   # connect the dots with lines + crosses (lines only: `lines`)
  title 'free blocks'
```

### Multiple lines, with data `x, line₁y, line₂y, ...`

```pl
# Input format:
#
#     x l1 l2 l3
#     1 2  3  4
#     2 3  4  5
#     3 4  5  6

set key autotitle columnheader     # use column headers as line titles

plot for [i=2:4] 'data.txt' \      # for: each column (1-based) from 2 to 4
  using 1:i \
  with linespoints
```

### Multiple lines

The `columnheader` is used to get the title, but the problem is that consumes the row, so it can't be gathered from a data row.

#### with dataset header and data `x, y`

Input format:

```pl
line1
2 3
3 4
4 5

line2
2 4
3 5
4 6
```

```pl
plot for [i=0:1] 'data.txt' \    # for: each index (0-based) from 0 to 1
  index i \
  using 1:2 \
  with linespoints \
  title columnheader(1)
```

Reference: https://stackoverflow.com/questions/12818797/how-to-plot-several-datasets-with-titles-from-one-file-in-gnuplot.

#### with data `lineN, x, y`, simple but less pretty

Input format:

```pl
l1 2 3
l1 3 4
l1 4 6


l2 2 4
l2 3 5
l2 4 4
```

This solution doesn't print titles (top right), instead, it uses labels (located on the rightmost point of the line).

The first plot prints the lines, without title; the second prints the labels, on the last value of each set.

```pl
plot \
  'data.txt' using 2:3 notitle with linespoints, \
  'data.txt' using 2:3:1 every ::2::2 notitle with labels
```

`every` performs a periodic sampling; syntax:

    every {<point_incr>}
            {:{<block_incr>}
              {:{<start_point>}
                {:{<start_block>}
                  {:{<end_point>}
                    {:<end_block>}}}}}

`every ::2::2` -> start_point=2, end_point=2

References:

- https://stackoverflow.com/questions/32603146/gnuplot-plot-multiple-lines-from-single-file-and-make-title-at-end-from-column
- https://stackoverflow.com/questions/12818797/how-to-plot-several-datasets-with-titles-from-one-file-in-gnuplot

#### with data `lineN, x, y`, prettier but more complex (requires Perl), with conveniences

Input format:

```pl
l1 2 3
l1 3 4
l1 4 6
l2 2 4
l2 3 5
l2 4 4
```

Solution with titles, but requires Perl; some conveniences are applied (automatic count sets; addition of blank lines):

```sh
local datasets_count=$(awk '{ print $1 }' data.txt | grep -v '^$' | uniq | wc -l)
local processed_file=$(mktemp)

perl -pae 'if (@F[0] ne $last_test_name) { print "\n\n@F[0]\n" }; $last_test_name = @F[0]' data.txt > "$processed_file"

gnuplot --persist -e "plot for [i=0:$(( datasets_count - 1 ))] '$processed_file' index i using 2:3 with linespoints title columnheader"
```

## Maths

```pl
# Stats (min, max, etc.)
# It's possible to use (using) two columns, which yields stats in the format *_max_x and *_max_y
# Without nooutput, stats are also printed.

stats 'test.txt' using 2:3 nooutput
y_padding = (STATS_max - STATS_min) * 0.04
set yrange [STATS_min - y_padding:STATS_max + y_padding]

stats [*:*][*:*] 'test.txt' using 3 nooutput
y2_padding = (STATS_max - STATS_min) * 0.04
set y2range [STATS_min - y_padding:STATS_max + y2_padding]
```

## Aesthetics

```pl
# Smoothen

plot 'test.txt' using 1:2 smooth sbezier with linespoints

# Break in different blocks
# Each blocks is separated by one empty line
# Note that even for lines covering the whole plot, the line is broken where the index separator is.

plot 'test.txt' index 0 using 1:2 with lines linewidth 2,\
     'test.txt' index 0:1 using 1:2 with lines linewidth 4

# Fill below line

plot 'test.txt' using 1:2 with filledcurves x1

# Fill sections below line - basic (doesn't work with dates)
# Use a filter
# Set filter giving invalid value when outside the range
# Doesn't work with dates

filter(x, min, max) = (x > min && x < max) ? x : 1/0
plot 'test.txt' using (filter($1, 5, 8)):2 with filledcurves x1,\
     'test.txt' using (filter($1, 15, 22)):2 with filledcurves x1,\
     'test.txt' using 1:2 with lines linewidth 4

# Fill sections below line, using dates
# Must use timecolumn() to convert the (time) column into the internal datetime format

set xdata time
set timefmt '%Y-%m-%d'
filter(x, min) = (x > strptime('%Y-%m-%d', min)) ? x : 1/0
plot 'test.txt' using (filter(timecolumn(1), '2019-01-10')):2 with filledcurves x1,\
     'test.txt' using 1:2 with lines linewidth 3

# Set margins in percentage (doesn't work with two y scales)
#
set offsets graph 0, 0, 0.05, 0.05
```

## More complex cases

### Two lines, with different scales

```pl
set yrange [65:80]
set y2range [5:25]
set y2tics			        # add the scale on the right
set ytics nomirror	    # don't mirror the left scale ticks on the right scale
plot '$diet_file' using 1:2 w linespoints title 'graph1',\
     '$diet_file' using 1:3 w linespoints title 'graph2' axes x1y2
```

## Ready scripts

### Plot base common global status values

```sh
# highlight critical interval using vertical bars

data_start='2015-07-31 11:38:00'
highlight_start='2015-07-31 11:39:01'
highlight_end='2015-07-31 11:39:22'
data_end='2015-07-31 11:40:30'

data_start_secs=`date -d "$data_start" +"%s"`
highlight_start_secs=`date -d "$highlight_start" +"%s"`
highlight_end_secs=`date -d "$highlight_end" +"%s"`
data_end_secs=`date -d "$data_end" +"%s"`

rel_highlight_start=`expr $highlight_start_secs - $data_start_secs`
rel_highlight_end=`expr $highlight_end_secs - $data_start_secs + 1`
rel_data_end=`expr $data_end_secs - $data_start_secs + 1`

rm -f *.png

for var in \
  Qcache_free_blocks \
  Qcache_free_memory \
; do
  mysql mysql_stats -BNe "
    SELECT TIMESTAMPDIFF(SECOND, '$data_start', created_at), value FROM global_statuses
    WHERE variable = '$var' AND (created_at BETWEEN '$data_start' AND '$data_end')
  " > /tmp/$var.txt

  gnuplot << PLT
    set terminal pngcairo size 1600,800;
    set output '$var.png';
    set format y "%s %c";

    set arrow from $rel_highlight_start, graph 0 to $rel_highlight_start, graph 1 nohead
    set arrow from $rel_highlight_end, graph 0   to $rel_highlight_end, graph 1 nohead

    plot '/tmp/$var.txt' using 1:2 with lines title '$var';
PLT
done
```

### Plot global status values, using deltas

```sh
# - compute deltas
# - highlight critical interval using vertical bars
#
# Putting all in a group would be really convenient, but the delta functionality
# and even setting old_v would have to be indexed.

data_start='2015-07-31 11:38:00'
highlight_start='2015-07-31 11:39:01'
highlight_end='2015-07-31 11:39:22'
data_end='2015-07-31 11:40:30'

data_start_secs=`date -d "$data_start" +"%s"`
highlight_start_secs=`date -d "$highlight_start" +"%s"`
highlight_end_secs=`date -d "$highlight_end" +"%s"`
data_end_secs=`date -d "$data_end" +"%s"`

rel_highlight_start=`expr $highlight_start_secs - $data_start_secs`
rel_highlight_end=`expr $highlight_end_secs - $data_start_secs + 1`
rel_data_end=`expr $data_end_secs - $data_start_secs + 1`

rm -f *.png

for var in \
  Innodb_buffer_pool_pages_flushed \
  Innodb_os_log_written \
; do
  mysql mysql_stats -BNe "
    SELECT TIMESTAMPDIFF(SECOND, '$data_start', created_at), value FROM global_statuses
    WHERE variable = '$var' AND (created_at BETWEEN '$data_start' AND '$data_end')
  " > /tmp/$var.txt

  gnuplot << PLT
    set terminal pngcairo size 1600,800;
    set output '$var.png';
    set format y "%s %c";

    set arrow from $rel_highlight_start, graph 0 to $rel_highlight_start, graph 1 nohead
    set arrow from $rel_highlight_end, graph 0   to $rel_highlight_end, graph 1 nohead

    delta_v(x) = (vD = x - old_v, old_v = x, vD)
    old_v = NaN

    plot '/tmp/$var.txt' using 1:(delta_v(\$2)) with lines title 'delta_$var';
PLT
done
```

### Plot multiple global status values, in a single chart

```sh
# - multiple lines in a single char
# - no autoscale
#
# VARIABLES MUST BE SORTED BY NAME WHEN WRITING THEM!

min_created_at=$(mysql mysql_stats -BNe "SELECT MIN(created_at) FROM global_statuses")

mysql mysql_stats -BNe "
  SELECT TIMESTAMPDIFF(SECOND, '$min_created_at', created_at), value FROM global_statuses
  WHERE variable IN (
    'Created_tmp_disk_tables',
    'Created_tmp_tables'
  )
  ORDER BY created_at, variable
" > /tmp/group_data.txt

gnuplot --persist << PLT
  set format y "%s %c";
  unset autoscale;
  set autoscale x;
  set autoscale ymax;

  plot \
    '/tmp/group_data.txt' every 2::0 using 1:2 with lines title 'Created_tmp_disk_tables', \
    '/tmp/group_data.txt' every 2::1 using 1:2 with lines title 'Created_tmp_tables', \
  ;
PLT
```
