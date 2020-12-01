# RSpec

- [RSpec](#rspec)
  - [Structure examples](#structure-examples)
  - [Setup](#setup)
    - [Add helpers to describes](#add-helpers-to-describes)
  - [Mocks](#mocks)
    - [Matchers](#matchers)
    - [Matching arguments](#matching-arguments)
    - [Responses](#responses)
  - [Testing modules](#testing-modules)
  - [Custom matcher](#custom-matcher)

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

## Setup

On top of each test file, add `require 'rspec'`, which will load `spec/spec_helper.rb`.

### Add helpers to describes

```ruby
RSpec.configure do |config|
  # Make `:my_key` a shorthand for `my_key: true`
  #
  config.treat_symbols_as_metadata_keys_with_true_values = true

  # Static inclusion.
  #
  config.include MyHelper, :include_my_helper
  config.extend  MyHelper, :extend_my_helper

  # Conditional inclusion, based on the described class.
  #
  config.include MailerSpecHelper, example_group: do
    describes: -> { |described| described < ActionMailer::Base }
  end
end

describe DividedPayment, :include_my_helper do
  # the helper is NOT available here!

  it "should do something" do
    # the helper is available here!
  end
end

describe DividedPayment, :extend_my_helper do
  # the helper is available here!

  it "should do something" do
    # the helper is NOT available here!
  end
end
```

## Mocks

### Matchers

See [reference](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/type-matchers).

 | Target       | Methods                                              | Notes                |
 | ------------ | ---------------------------------------------------- | -------------------- |
 | Class (type) | `be_a`, `be_an`, `be_a_kind_of`, `be_an_instance_of` |                      |
 |              | `be` `<|<=|==|=>|>` `expected`                       |                      |
 |              | `be_within(span).of(reference)`                      | Time/Floats          |
 |              | `be === expected`                                    |                      |
 |              | `eq(expected)`                                       |                      |
 |              | `change { instance.attribute }.from(start).to(end)`  |                      |
 |              | `change { instance.attribute }.by(amount)`           |                      |
 |              | `match(pattern)`                                     | String also accepted |
 |              | `match_array(array)`                                 |                      |
 |              | `be_false`, `be_true`, `be_falsey`, `be_truthy`      |                      |
 |              | `be_empty`                                           |                      |

### Matching arguments

See [reference](https://relishapp.com/rspec/rspec-mocks/docs/setting-constraints/matching-arguments).

| Argument                   | Methods            | Notes                                     |
| -------------------------- | ------------------ | ----------------------------------------- |
| Regexp/===                 | with(/expr/)       | Anything supporting case equality (`===`) |
| Instance of specific class | instance_of(klazz) |                                           |

### Responses

See [reference](https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/configuring-responses).

- `and_return`
- `and_raise`
- `and_throw`
- `and_yield`
- `and_call_original`
- `and_wrap_original`

Conditional stubbing:

```ruby
allow(repository).to receive(:remote).with('origin').and_return('git@github.com:saveriomiroddi/goby-dev')
allow(repository).to receive(:remote).with('upstream').and_return('git@github.com:goby-lang/goby')

# Better for modifying the way the method is called, rather than selective stubbing
#
allow(API).to receive(:solve_for).and_wrap_original { |m, *args| m.call(*args).first(5) }
```

Return different values for each invocation:

```ruby
# Returns 1, then 2, then 3.
#
expect_any_instance_of(AWSInfo).to receive(:burst_balance).and_return(1, 2, 3)
```

Arbitrary handling (references: [README](https://github.com/rspec/rspec-mocks#arbitrary-handling) and [Rubydoc](https://rubydoc.info/gems/rspec-mocks#arbitrary-handling)):

```ruby
# WATCH OUT! When using :expect_any_instance_of, the first arg is the instance.
# If a `with()` expectation is added, it's still verified.
#
expect(double).to receive(:msg) do |*args|
  expect(args.size).to eq(7)
end
```

## Testing modules

```ruby
let(:helper) { Class.new { extend ModuleToTest } }
```

## Custom matcher

```ruby
class LooksLike < String
  def failure_message
    "#{ @me } different from #{ @another }"
  end

  def matches?(another)
    @me      = some_operation(self)
    @another = some_operation(another)

    @me == @another
  end
end

def look_like(string)
  LooksLike.new(string)
end

expect("abcd").to look_like("abcd")
```