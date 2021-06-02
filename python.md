# Python

- [Python](#python)
  - [General syntax](#general-syntax)
    - [Strings/Printing](#stringsprinting)
    - [Data types](#data-types)
    - [Classes](#classes)
    - [Collections](#collections)
      - [Lists](#lists)
      - [Dicts](#dicts)
    - [Iteration/Comprehension](#iterationcomprehension)
    - [Functions](#functions)
    - [Operators](#operators)
    - [Time stuff](#time-stuff)
  - [APIs/Stdlib](#apisstdlib)
    - [file](#file)
    - [logging (import)](#logging-import)
    - [pprint (pretty printing) (import)](#pprint-pretty-printing-import)
    - [Debugging](#debugging)
    - [I/O](#io)
  - [Snippets](#snippets)
    - [Read a CSV from stdin, and compute averages](#read-a-csv-from-stdin-and-compute-averages)

## General syntax

### Strings/Printing

```python
f'{threads},{sum(run_times) / len(run_times)}' # interpolate strings ("f-strings", v3.6+)
```

### Data types

Parse strings:

```python
int(str)
float(str)
```

### Classes

```python
type(instance).__name__      # get class name
isinstance(instance, class)  # check instance class
```

### Collections

```python
a, _, c = collections       # unpack tuple/list

sorted(iterable)            # return a sorted collection
sorted(dict.items())        # iteratable (k, v) of a dict, sorted by key

next(iterable)              # return the next entry; can be used to skip one
```

#### Lists

```python
len(list)                   # size/length/count of a list
list.append(value)          # append an item
list[-1] if list else None  # safe get
not list                    # idiomatic check if a list is empty
```

#### Dicts

```python
dict[key]                 # perform a lookup; if the key doesn't exist, an error is raised
dict.setdefault(key, default)  # return a value, setting the given default if not present
key in dict               # check if dict contains key
dict.get(key[, default])  # perform a lookup; if default is not provided and the key doesn't exist, an error is raised
dict.pop(key[, default])  # delete a key; if default is not provided and the key doesn't exist, an error is raised
```

### Iteration/Comprehension

```python
for i in range(r)         # iterate in [0, 5)
for i in range(l, r, s)   # iterate with given (l)eft limit and (s)tep; the step can be negative (if so, make l > s).

for e in list             # iterate over a list

for k, v in dict.items()  # iterate over a dict
for k in dict             # iterate over a dict keys
```

Comprehension:

```python
regions = [(r.a, r.b) for r in folded_regions]    # example
[a*10 for a in [1, 2] if a == 2]                  # example with conditional; elements not passing the test won't be included
```

### Functions

```python
def myfunction(first_arg, default_arg_1 = 'd1', default_arg_2 = 'd2'):  # definition
myfunction('fa', default_arg_2 = 'd2a')                                 # invocation
myfunction(default_arg_2 = 'd2a', first_arg = 'fa')                     # also valid invocation

# Return tuples
def myfunction():
  return 1, 2
a, b = myfunction()
```

### Operators

Math:

- `//`: floor of int division (like ruby, ie. `-3 // 2 == -2`)

Booleans:

- `not`/`or` have precedence as in other languages
- `and`/`or` do short-circuit

### Time stuff

```python
# Get the current time as a zone-aware object
#
from datetime import datetime, timezone
now = datetime.now(timezone.utc)
```

## APIs/Stdlib

### file

```python
os.path.exists(filename) and os.path.isfile(fiename)    # check if a file exists
with open(filename, 'w') as file:                       # open file; no mode will open for read
  data = file.read()
  file.write(new_data)

content = open(filename, 'r').read()                    # one-liner; relies on exit to close the handle
```

### logging (import)

```python
try:
  fail()
except:
  logging.exception("Error!")    # logs the full stacktrace, with the specified header
```

### pprint (pretty printing) (import)

```python
pprint.pprint(object)            # good enough in the default configuration
```

### Debugging

Add to any line:

```python
import pdb; pdb.set_trace()
```

### I/O

Access and iterate sys.stding

```python
import sys
next(sys.stdin)                          # skip line
for line in map(str.rstrip, sys.stdin):  # strip the trailing whitespace
```

## Snippets

### Read a CSV from stdin, and compute averages

Doesn't use the `csv` api, for simplicity. Requires v3.5+.

Sample input:

```csv
theads,run_number,run_time
4,0,2.22
4,1,2.33
8,0,1.11
```

```python
import sys

all_run_times={}

next(sys.stdin)

for run_data in map(str.rstrip, sys.stdin):
  threads, _, run_time = run_data.split(',')
  threads_run_times = all_run_times.setdefault(int(threads), [])
  threads_run_times.append(float(run_time))

for threads, run_times in sorted(all_run_times.items()):
  print(f'{threads},{sum(run_times) / len(run_times)}')
```
