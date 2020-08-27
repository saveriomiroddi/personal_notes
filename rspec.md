# RSpec

- [RSpec](#rspec)
  - [Mocks](#mocks)
    - [??? SECTION ???](#-section-)
    - [Arbitrary handling](#arbitrary-handling)

## Mocks

### ??? SECTION ???

```ruby
`be_a`, `be_an`, `be_a_kind_of`, `be_an_instance_of`
```

### Arbitrary handling

References [README](https://github.com/rspec/rspec-mocks#arbitrary-handling) and [Rubydoc](https://rubydoc.info/gems/rspec-mocks#arbitrary-handling).

```ruby
# WATCH OUT! When using :expect_any_instance_of, the first arg is the instance.
#
expect(double).to receive(:msg) do |*args|
  expect(args.size).to eq(7)
end
```

