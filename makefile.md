# Makefile

- [Makefile](#makefile)
  - [Sample breakdown](#sample-breakdown)
    - [Variables](#variables)
    - [Targets/Symbols](#targetssymbols)
    - [Shell commands execution](#shell-commands-execution)
    - [If/then/else](#ifthenelse)

## Sample breakdown

The makefile name can be `makefile` or `Makefile`

This makefile assumes the file `main.c` and the dependencies `module.c`/`module.h`.

### Variables

Variables are conventonally upper case, and can be referenced with curly braces; they are referenced with `${VAR_NAME}`; missing vars produce an empty string.

- `:=` : "simple assignment"; it's evaluated only once;
- `=`  : "recursive assigment"; it's re-evaluated every time (so changing the value is recognized);
- `?=` : assign if not already done;
- `+=` : append;

Variables can also be used in the dependencies part.

```makefile
CC := gcc
OPTIONS := -O2 -g -Wall
INCLUDES := -I .
LIST_FILES := ls -l
```

### Targets/Symbols

Targets which are filenames, are compiled only if the file changed; non-filenames are always compiled.
A target is a filename (or an arbitrary name), followed by the dependencies; each target is executed in a separate subshell.

An action can be prefixed with `-` to ignore errors (see clean).

The prefix `@` doesn't echo the action.

`%` : wildcard.

Variables:

- `$@` : (full target name of the current target)
- `$?` : (returns the dependencies that are newer than the current target)
- `$*` : (returns the text that corresponds to % in the target)
- `$<` : (name of the first dependency)
- `$^` : (name of all the dependencies with space as the delimiter)

```makefile
# Specify that those targets are not filenames - in this case, this is optional.
# .PHONY can accept multiple values.
#
.PHONY all

# `all` is the default target
#
all: main.o module.o
	${CC} ${OPTIONS} main.o module.o -o target_bin

# The following are generalized in the target with wildcard.
#
# main.o: main.c module.h
# 	${CC} ${OPTIONS} ${INCLUDES} -c main.c
# module.o: module.c module.h
# 	${CC} ${OPTIONS} ${INCLUDES} -c module.c

%.o: %.c %.h
	${CC} ${OPTIONS} ${INCLUDES} -c $*.c
```

### Shell commands execution

`$(shell <command>)`: Executes a shell command (eg. `$(shell ls)`); it can also be assigned to a variable (eg. `VAR := $(...)`).

A variable can be used a command (it seems that it must be the only content of the action) - see this example.

```makefile
.PHONY: list
list:
	@$(LIST_FILES)

.PHONY: clean
clean:
	rm -f *.o
	-rm target_bin
```

### If/then/else

- `ifeq($(VARIABLE),value)`
- `ifneq`

```makefile
# Also shows recursive rules invocation (`$(MAKE)`)
#
# Also shows checking if a variable is defined, and raising errors.
#
.PHONY: install
install:
ifeq($(GOBY_BINPATH),)
	go install $(RELEASE_OPTIONS) .
else
ifndef GOBY_LIBPATH
	$(error GOBY_BINPATH requires GOBY_LIBPATH to be set)
endif
	$(MAKE) build
	mkdir -p ${DESTDIR}${GOBY_BINPATH}
	mv -v goby ${DESTDIR}${GOBY_BINPATH}
	mkdir -p ${GOBY_LIBPATH}
	cp -R lib ${GOBY_LIBPATH}
endif
```
