# GitHub

- [GitHub](#github)
  - [Ruby CI action](#ruby-ci-action)
    - [Caching](#caching)
    - [Minimal example](#minimal-example)

## Ruby CI action

Reference: https://docs.github.com/en/actions/guides/building-and-testing-ruby.

Add a file named `.github/workflows/ci.yml`:

```yml
# Note that this is not legal YAML (!)
#
name: Ruby CI

# https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#on
#
on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]
  #
  # Other examples:
  #
  # page_build:
  # release:
  #   types: # This configuration does not affect the page_build event above
  #     - created

# Simplified form (also single string accepted)
#
# on: [push, pull_request]
#

jobs:
  test:
    # This doesn't guarantee a specific version, may run on the beforelast version.
    #
    runs-on: ubuntu-latest
    # https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstrategy
    #
    strategy:
      fail-fast: false # default: true
      matrix: # optional
        os: [ubuntu, macos]
        ruby-version: [head, 3.0, 2.9, 2.8, 2.7, 2.6, 2.5]
    # `debug` is only to show the functionality - it's not included above.
    #
    continue-on-error: ${{ endsWith(matrix.ruby, 'head') || matrix.ruby == 'debug' }}
    # See caching section
    steps:
    - uses: actions/checkout@v2
    - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
    - name: Set up Ruby ${{ matrix.ruby-version }}
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ matrix.ruby-version }}
        bundler-cache: true
    - name: Install dependencies
      run: bundle install
    - name: Run tests
      run: bundle exec rspec
```

### Caching

Base caching:

```yml
- uses: actions/cache@v2
  with:
    path: |
      ~/.cargo/registry
      ~/.cargo/git
      target
    key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
```

Ruby bundler caching is not trivial, so there's a specific action:

```yml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: ...
    bundler-cache: true
```

Other examples: https://github.com/actions/cache/blob/main/examples.md.

### Minimal example

```yml
name: CI

on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run shellcheck
      run: ci/run_shellcheck.sh
```
