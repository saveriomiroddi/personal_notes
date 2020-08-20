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
# WATCH OUT! Odd behavior (at least, in TS). See issue https://git.io/JJp8c.
# If there are multiple expectations, split them in multiple blocks, and use allow for the outer one;
# also, consider that errors will make the test pass (!).
#
expect(double).to receive(:msg) do |arg|
  expect(arg.size).to eq(7)
end

# Working
allow(double).to receive(:msg) do |arg|
  expect(arg.size).to eq(7)
end
allow(double).to receive(:msg) do |arg|
  expect(arg).to be_a(Array)
end
```

