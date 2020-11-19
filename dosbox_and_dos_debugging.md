# DOSBox and DOS debugging

## DOSBox sample execution

```sh
dosbox -c 'MOUNT C "."' -c 'C:' -c 'DEBUG SAMPLE.COM'
```

## DEBUG.COM commands

- `U [address]`          disassemble
- `T [=address] [count]` trace (execute instructions and stop); with address, it stops **past** it
- `A [address]`          assemble
- `D`                    dump
- `R [register [value]]` print/change registers
