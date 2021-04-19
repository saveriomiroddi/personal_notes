# C

- [C](#c)
  - [Basics](#basics)
    - [Base program and functions](#base-program-and-functions)
      - [printf](#printf)
  - [Conveniences](#conveniences)
    - [Print a struct instances's bytes](#print-a-struct-instancess-bytes)
    - [Print a stack trace on segfault](#print-a-stack-trace-on-segfault)
  - [Make](#make)
  - [Configure/Compiler (gcc)](#configurecompiler-gcc)
  - [Library issues](#library-issues)
    - [Error `glibconfig.h: No such file or directory`](#error-glibconfigh-no-such-file-or-directory)
    - [Error `cannot find install-sh, install.sh, or shtool in ...`](#error-cannot-find-install-sh-installsh-or-shtool-in-)

## Basics

### Base program and functions

```c
#include <fcntl.h>    // open(2), O_RDONLY, AT_FDCWD
#include <stdio.h>    // printf(3)
#include <stdlib.h>   // exit(3), EXIT_FAILURE
#include <unistd.h>   // read(2), close(2)

int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "Invalid number of command line arguments.\n");
    exit(EXIT_FAILURE);
  }

  printf("123");

  system("ls -l /tmp");

  int length = read(fd, buffer, EVENT_BUF_LEN);
  close(fd);
}
```

#### printf

- `lu/ld` long unsigned/signed

## Conveniences

### Print a struct instances's bytes

```c
for (int i = 0; i < sizeof(struct fanotify_event_metadata); i++)
  printf("%02x ", ((unsigned char*)metadata)[i]);
printf("\n");
```

### Print a stack trace on segfault

Requires enabling the debug symbols:

```c
#include <execinfo.h>
void sav_handler(int sig);

void sav_handler(int sig) {
  void *array[10];

  // get void*'s for all entries on the stack
  size_t size = backtrace_symbols(array, 10);

  fprintf(stderr, "Error: signal %d:\n", sig);
  backtrace_symbols_fd(array, size, STDERR_FILENO);

  exit(1);
}

int main(int argc, char **argv) {
  signal(SIGSEGV, sav_handler); // install the handler
  raise(SIGSEGV);
}
```

Send the result to addr2line:

```sh
a.out 2>&1 | perl -ne '/\(\+(0x\w+)\)/ && print("$1 ")' | xargs addr2line -e a.out
```

## Make

```sh
# Note that 'make' depends on the product of './configure'.
#
make clean           # Remove `make` output
make distclean       # Remove `configure`+`make` output
```

## Configure/Compiler (gcc)

- `-static`         : compile statically
- `-Dmacro[=value]` : define a macro (e.g. for `ifdef`); the option is cumulative

## Library issues

### Error `glibconfig.h: No such file or directory`

See https://github.com/dusty-nv/jetson-inference/issues/6:

```sh
sudo ln -s /usr/lib/x86_64-linux-gnu/glib-2.0/include/glibconfig.h /usr/include/glib-2.0/
```

or try this:

```sh
gcc pkg-config --cflags glib-2.0 test.c pkg-config --libs glib-2.0
```

or suggested include path (maybe should be `/usr/lib/x86_64-linux-gnu/glib-2.0/include/glibconfig.h`?):

```
${workspaceFolder}/** /usr/include/glib-2.0/**
```

### Error `cannot find install-sh, install.sh, or shtool in ...`

Execute:

```sh
autoreconf -vif
# follow up with ./configure etc.
```