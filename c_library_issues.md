# C Library Issues

- [C Library Issues](#c-library-issues)
  - [Fix error `glibconfig.h: No such file or directory`](#fix-error-glibconfigh-no-such-file-or-directory)

## Fix error `glibconfig.h: No such file or directory`

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


