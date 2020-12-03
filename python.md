# Python

- [Python](#python)
  - [General syntax](#general-syntax)
    - [Classes](#classes)
    - [Collections](#collections)
      - [Lists](#lists)
      - [Dicts](#dicts)
    - [Functions](#functions)
    - [Operators](#operators)
    - [Time stuff](#time-stuff)
  - [APIs/Stdlib](#apisstdlib)
    - [file](#file)
    - [logging (import)](#logging-import)
    - [pprint (pretty printing) (import)](#pprint-pretty-printing-import)
    - [Debugging](#debugging)

## General syntax

### Classes

```python
type(instance).__name__      # get class name
isinstance(instance, class)  # check instance class
```

### Collections

#### Lists

```python
len(list)                   # size/length/count of a list
list.append(value)          # append an item
list[-1] if list else None  # safe get
not list                    # idiomatic check if a list is empty

# Comprehension

regions = [(r.a, r.b) for r in folded_regions]    # example
[a*10 for a in [1, 2] if a == 2]                  # example with conditional; elements not passing the test won't be included
```

#### Dicts

```python
dict[key]                 # perform a lookup; if the key doesn't exist, an error is raised
key in dict               # check if dict contains key
dict.get(key[, default])  # perform a lookup; if default is not provided and the key doesn't exist, an error is raised
dict.pop(key[, default])  # delete a key; if default is not provided and the key doesn't exist, an error is raised

for k, v in dict.items()  # iterate over a dict
for k in dict             # iterate over a dict keys
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

```python
# not/or have precedence as in other languages
# and/or do short-circuit
```

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
