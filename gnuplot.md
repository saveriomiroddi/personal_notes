# gnuplot

- [gnuplot](#gnuplot)
  - [Base elements](#base-elements)
  - [Single plotting](#single-plotting)
    - [Single line, formatted with dates](#single-line-formatted-with-dates)
    - [Two lines, with different scales](#two-lines-with-different-scales)
  - [General options](#general-options)
  - [Ready scripts](#ready-scripts)
    - [Plot base common global status values](#plot-base-common-global-status-values)
    - [Plot global status values, using deltas](#plot-global-status-values-using-deltas)
    - [Plot multiple global status values, in a single chart](#plot-multiple-global-status-values-in-a-single-chart)

## Base elements

```gnuplot
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

# TRY: use whatever wanted as Y
plot ... using 4:xtic(1)

# Debug (`-` signifies stdout)
set print '-'
print ...

# Stats (min, max, etc.)
# It's possible to use (using) two columns, which yields stats in the format *_max_x and *_max_y
# Without nooutput, stats are also printed.
stats 'test.txt' using 2:3 nooutput
y_padding = (STATS_max - STATS_min) * 0.04
set yrange [STATS_min - y_padding:STATS_max + y_padding]

stats [*:*][*:*] 'test.txt' using 3 nooutput
y2_padding = (STATS_max - STATS_min) * 0.04
set y2range [STATS_min - y_padding:STATS_max + y2_padding]

# CSV
Separator ","

# Batch mode: displays the diagram and exits
gnuplot --persist #{file.path}

# Set margins in percentage (doesn't work with two y scales)
set offsets graph 0, 0, 0.05, 0.05
```

## Single plotting

### Single line, formatted with dates

```gnuplot
Sample: "06-12 10\n06-13 11"

# using rows `1` as x and `2` as y
# connecting the sample dots with lines + crosses (lines only: `lines`)
# with title `free blocks`
# parse the x as date, in a specific format

set xdata time
set timefmt "%m-%d"
plot "data.txt" using 1:2 w linespoints title 'free blocks'
```

### Two lines, with different scales

```gnuplot
set yrange [65:80]
set y2range [5:25]
set y2tics			        # add the scale on the right
set ytics nomirror	    # don't mirror the left scale ticks on the right scale
plot '$diet_file' using 1:2 w linespoints title 'graph1',\
     '$diet_file' using 1:3 w linespoints title 'graph2' axes x1y2
```

## General options

```gnuplot
set datafile separator ','	# accept a CSV data file
pause mouse close			      # show the diagram, and wait for close before exit; requires return to be tapped - this can be worked around by
					                  # `echo`ing a newline to stdin.
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

  cat << YAYGNUPLOT | gnuplot
    set terminal pngcairo size 1600,800;
    set output '$var.png';
    set format y "%s %c";

    set arrow from $rel_highlight_start, graph 0 to $rel_highlight_start, graph 1 nohead
    set arrow from $rel_highlight_end, graph 0   to $rel_highlight_end, graph 1 nohead

    plot '/tmp/$var.txt' using 1:2 with lines title '$var';
YAYGNUPLOT
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

  cat << YAYGNUPLOT | gnuplot
    set terminal pngcairo size 1600,800;
    set output '$var.png';
    set format y "%s %c";

    set arrow from $rel_highlight_start, graph 0 to $rel_highlight_start, graph 1 nohead
    set arrow from $rel_highlight_end, graph 0   to $rel_highlight_end, graph 1 nohead

    delta_v(x) = (vD = x - old_v, old_v = x, vD)
    old_v = NaN

    plot '/tmp/$var.txt' using 1:(delta_v(\$2)) with lines title 'delta_$var';
YAYGNUPLOT
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

cat << YAYGNUPLOT | gnuplot -p
  set format y "%s %c";
  unset autoscale;
  set autoscale x;
  set autoscale ymax;

  plot \
    '/tmp/group_data.txt' every 2::0 using 1:2 with lines title 'Created_tmp_disk_tables', \
    '/tmp/group_data.txt' every 2::1 using 1:2 with lines title 'Created_tmp_tables', \
  ;
YAYGNUPLOT
```
