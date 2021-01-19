# C

- [C](#c)
  - [Basics](#basics)
    - [Base program and functions](#base-program-and-functions)
  - [Conveniences](#conveniences)
    - [Print a struct instances's bytes](#print-a-struct-instancess-bytes)
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

## Conveniences

### Print a struct instances's bytes

```c
for (int i = 0; i < sizeof(struct fanotify_event_metadata); i++)
  printf("%02x ", ((unsigned char*)metadata)[i]);
printf("\n");
```

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