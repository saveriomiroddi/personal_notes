# Travis

- [Travis](#travis)
  - [Allowed failures](#allowed-failures)

## Allowed failures

The `fast_finish` marks the build as completed without waiting for allowed to fail jobs.

```yaml
rvm:
  - 2.7
  - ruby-head
matrix:
  fast_finish: true
  allow_failures:
    - rvm: ruby-head
```
