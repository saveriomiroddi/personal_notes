# RSpec

- [RSpec](#rspec)
  - [Structure examples](#structure-examples)
  - [Mocks](#mocks)
    - [Matchers](#matchers)
    - [Matching arguments](#matching-arguments)
    - [Arbitrary handling](#arbitrary-handling)
  - [Testing modules](#testing-modules)

## Structure examples

```ruby
describe Game
  describe "#start"
    it "sends a welcome message"
    it "prompts for the first guess"
  describe "#guess"
    context "with no matches"
      it "sends a mark with ''"
    context "with 1 number match"
      it "sends a mark with '-'"
    context "with 1 exact match"
      it "sends a mark with '+'"

describe Stack
  describe "when full"
    describe "when it receives push"
      it "should raise an error"
  describe "when almost full (one less than capacity)"
    describe "when it receives push"
      it "should be full"

describe MessagesController
  describe "POST create" do
    it "creates a new message"
    it "saves the message"
    context "when the message saves successfully"
      it "sets a flash[:notice] message"
      it "redirects to the Messages index"
    context "when the message fails to save"
      it "assigns @message"
      it "renders the new template"
```

## Mocks

### Matchers

See [reference](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/type-matchers).

 | Target       | Methods                                              |
 | ------------ | ---------------------------------------------------- |
 | Class (type) | `be_a`, `be_an`, `be_a_kind_of`, `be_an_instance_of` |

### Matching arguments

See [reference](https://relishapp.com/rspec/rspec-mocks/docs/setting-constraints/matching-arguments).

| Argument   | Methods      | Notes                                     |
| ---------- | ------------ | ----------------------------------------- |
| Regexp/=== | with(/expr/) | Anything supporting case equality (`===`) |

### Arbitrary handling

References [README](https://github.com/rspec/rspec-mocks#arbitrary-handling) and [Rubydoc](https://rubydoc.info/gems/rspec-mocks#arbitrary-handling).

```ruby
# WATCH OUT! When using :expect_any_instance_of, the first arg is the instance.
#
expect(double).to receive(:msg) do |*args|
  expect(args.size).to eq(7)
end
```

## Testing modules

```ruby
let(:helper) { Class.new { extend ModuleToTest } }
```
