# Ruby

- [Ruby](#ruby)
  - [General syntax](#general-syntax)
    - [Array literals](#array-literals)
    - [Runtime code evaluation](#runtime-code-evaluation)
    - [Block-oriented processing methods](#block-oriented-processing-methods)
    - [Ampersand prefix operator `&<object>`](#ampersand-prefix-operator-object)
    - [Heredoc](#heredoc)
    - [Numerical base conversion](#numerical-base-conversion)
  - [Special variables/Built-in constants](#special-variablesbuilt-in-constants)
  - [APIs/Stdlib](#apisstdlib)
    - [Array](#array)
    - [CGI/URI (encoding)](#cgiuri-encoding)
    - [Strings/encoding](#stringsencoding)
    - [Openstruct (ostruct)](#openstruct-ostruct)
  - [Handling processes](#handling-processes)
    - [Basic handling, via `IO.popen`](#basic-handling-via-iopopen)
    - [Using `IO.popen3`](#using-iopopen3)
    - [Live execution, via `Kernel#system`](#live-execution-via-kernelsystem)
    - [Backticks](#backticks)
  - [Misc](#misc)
    - [System](#system)
    - [Convert curl request to Ruby](#convert-curl-request-to-ruby)

## General syntax

### Array literals

```ruby
%i(foo bar)           # array of symbols
```

### Runtime code evaluation

Create a (namespaced) class runtime, inside a method:

```ruby
# Version using :class_eval and a heredoc string; the parameters passed enable source debugging.
#
ActiveTrigger.class_eval <<-'RUBY', __FILE__, __LINE__ + 1
  class Application < Rails::Application; end
'RUBY'

ActiveTrigger.const_set :Application, Class.new(Rails::Application) do; end
```

### Block-oriented processing methods

```ruby
# `#tap` allows modifying the object while returning the object
#
Customer.new.tap { |c| c.attribute = value; c.save! }

# `#then` (or `#yield_self`) returns instead the result of the block.
# Arguably, it's preferrable for a  pipeline - see https://zverok.github.io/blog/2018-01-24-yield_self.html:
#
url
  .then { |url| Faraday.get(url) }.body
  .then { |response| JSON.parse(response) }
  .dig('object', 'id')
  .then { |id| id || '<undefined>' }
  .then { |id| "server:#{id}" }

# opposed to:

"server:" + (JSON.parse(Faraday.get(construct_url)).dig('object', 'id') || '<undefined>')
```

### Ampersand prefix operator `&<object>`

`&<object>` is turned to  `object.to_proc`.

```ruby
myblock = ->(i) { 2 * i }
def mymethod(i); 2 * i; end

# &<block> => intuitive invocation
#
[1, 2, 3].map(&myblock)
[1, 2, 3].map { |i| 2 * i }

# &method(:<method>) => { |arg| <method>(arg) }
#
# method(:<method>) doesn't take any arguments, so it can't be turned into a `<method>(arg1, arg2...)` invocation.
#
[1, 2, 3].map(&method(:mymethod))
[1, 2, 3].map { |i| mymethod(i) }

# &:<method> => { |obj, *args| obj.send(<method>, *args) }
#
# Fits with :inject, but it's not clear what else could it apply to.
#
[1, 2, 3].inject(:+)
[1, 2, 3].inject { |i, j| i.send(:+), j }
```

### Heredoc

The closing token must be the only one on a line; preceding spaces are ignored (and a newline is appended).

With minus, allows delimiter to be anywhere:

```ruby
<<-EOF
  text
    EOF
```

With tilde, also allows delimiter to be anywhere:

```ruby
<<~EOF
  test
  EOF
```

### Numerical base conversion

Common conversions. Everything is a string; "int" represents a decimal.

Generic methods:

```ruby
# int -> a numerical string in the given base
# IMPORTANT! There's no left padding.
#
int.to_s(base)

# numerical string in the given base -> int
#
numerical_str.to_i(base)

# Numerical string to bytes; the directive is the base of the string; some directives:
# - `B*`: binary
# - `H*`: hex
#
[numerical_str].pack(directive)
```

Specific conversions:

```ruby
bytes.each_byte.map { |b| "%02X" % b }.join   # bytes → hex string (see printf link below)
bytes.each_byte.each_slice(2) { |i, j| puts "%016b" % (256 * i + j) } # bytes → binary, padded 16 bits big endian
bytes.each_byte.map(&:to_i)                   # bytes → array of ints
arr.map(&:chr).join                           # array of ints → bytes

"c".ord                                       # char  → int
int.chr                                       # int   → byte/char (ASCII-8)

hex.hex                                       # hex   → int (same as `to_i(16)`)
```

See https://idiosyncratic-ruby.com/49-what-the-format.html for `printf`-style formatting.

There is a gem, `bitarray`, however, it's not so much better than just using a string (bits = `"0" * 8`) for generic work, as it doesn't even support bit operations like `OR`.

Another gem, `bitset`, is very functional, but it's old, and needs to be checked.

## Special variables/Built-in constants

Updated up to: https://ruby-doc.org/stdlib-2.3.0/libdoc/English/rdoc/English.html

Entries marked with `ENG` need to `require 'English'` in order for the english version to be usable.

```ruby
$:   $LOAD_PATH                     # load path
$0   $PROGRAM_NAME                  # the name of the ruby script file
$*   $ARGV                          # [ENG] the command line arguments
$?   $CHILD_STATUS                  # [ENG] exit status of last executed child process
$$   $PID, $PROCESS_ID              # [ENG] interpreter's process ID

$~   $LAST_MATCH_INFO               # [ENG] MatchData instance for the last regexp match
$<n>                                # nth subexpression in the last match (same as $~[n])
$&   $MATCH                         # [ENG] string last matched by regexp
$+   $LAST_PAREN_MATCH              # [ENG] last match from the previous successful pattern match
$`   $PREMATCH                      # [ENG] string before the actual matched string of the previous successful pattern match.
$'   $POSTMATCH                     # [ENG] string after the actual matched string of the previous successful pattern match.

$!   $ERROR_INFO                    # [ENG] last error message
$@   $ERROR_POSITION                # [ENG] last error backtrace

$_   $LAST_READ_LINE                # [ENG] string last read by gets
$.   $NR, $INPUT_LINE_NUMBER        # [ENG] line number last read by interpreter

$,  $OFS, $OUTPUT_FIELD_SEPARATOR   # [ENG]
$;  $FS, $FIELD_SEPARATOR           # [ENG]
$/  $RS, $INPUT_RECORD_SEPARATOR    # [ENG] input record separator
$\  $ORS, $OUTPUT_RECORD_SEPARATOR  # [ENG] output record separator

$>  $DEFAULT_OUTPUT                  # [ENG]
$<  $DEFAULT_INPUT                   # [ENG]

$=  $IGNORECASE                      # [ENG] obsolete: case-insensitivity flag
```

```ruby
RUBY_PATCHLEVEL                     # patch level (Integer)
RUBY_VERSION                        # e.g. "2.7.0"
__dir__                              # directory of current file
```

## APIs/Stdlib

### Array

```ruby
arr = [0, 1, 2]

arr.fill(nil, arr.size...5)             # resize/extend (destructive) in arguably expressive form; returns the array; !!! watch out the `...` syntax !!!
arr[5] ||= nil                          # other resize/extend, in arguably less expressive form
```

### CGI/URI (encoding)

URI is already in scope.

```ruby
CGI::escape("&&&")                            # URL-encode a string: "%26%26%26"
URI.encode_www_form(p1: "&&&", "p2" => "!!!") # URL-encode params: "p1=%26%26%26&p2=%21%21%21"

CGI::escapeHTML('"html"')       # escape HTML

CGI.unescapeHTML("html")        # decode HTML entities; use only for basic cases, as it' not 100% complete (gem: https://github.com/threedaymonk/htmlentities)
HTMLEntities.new.decode("html")  # htmlentities gem
```

### Strings/encoding

```ruby
String.new(str, encoding: enc)         # !! Defaults to ASCII-8 encoding !!
```

### Openstruct (ostruct)

Convenient solution for converting a Hash to a recursive openstruct (including arrays), builder pattern-style:

```ruby
# Convert to a recursive openstruct (including arrays), builder pattern-style:
#
# Origin: https://stackoverflow.com/a/42520668.
#
def to_recursive_ostruct(node)
  case node
  when Hash
    node.each_with_object(OpenStruct.new) { |(key, value), ostruct| ostruct[key] = to_recursive_ostruct(value) }
  when Array
    node.map { |item| to_recursive_ostruct(item) }
  else
    node
  end
end
```

## Handling processes

The sections assume `require 'English'`, which aliases `$?` (`Process::Status`) to `$CHILD_STATUS`.

### Basic handling, via `IO.popen`

The stdout content is captured, so this is not appropriate for long operations.
Stderr content is instead printed immediately, unless `err: [:child, :out]` is passed, which causes it to be mixed with the stdout.

Invalid commands cause an error (`Errno::ENOENT`), while valid ones with failure exit status need to be detected (see below).

```ruby
stdout_content = IO.popen('sed "s/foo/bar/"', 'r+') do |io|
  io.puts "foo"
  io.close_write
  io.read # return value of the block
end

if !$CHILD_STATUS.success?
  exit $CHILD_STATUS.exitstatus
else
  print stdout_content # "bar\n"
end
```

A non-block form exists, which returns the `IO` object.

### Using `IO.popen3`

The output is not printed as well, so the same considerations as `IO.popen` apply.

```ruby
Open3.popen3([env,] command) do |stdin, stdout, stderr, wait_thread|
  stdout_content, stderr_content = stdout.read, stderr.read

  $stdout.puts(stdout_content) if stdout_content != ''
  $stderr.puts(stderr_content) if stderr_content != ''

  # `wait_thread.value` waits until completion
  #
  raise "Error (exit status: #{wait_thread.value.exitstatus})" if !wait_thread.value.success?

  stdout_content
end
```

### Live execution, via `Kernel#system`

This way, stdout/stderr content is printed straight away.

The exit status can be obtained directly via return value, or via `$CHILD_STATUS`.

```ruby
success = system(">&2 echo abc; sleep 2; false")

exit $CHILD_STATUS.exitstatus if !success
```

In order to pass env variables:

```ruby
# The hash key/values must be all strings.
#
system({'FOO' => 'BAR'}, 'echo $FOO') # => BAR
```

### Backticks

Backticks behave like `IO.popen`: The stdout content is captured, stderr content is instead printed immediately.

## Misc

### System

Current user run dir, on Linux:

```ruby
ENV.fetch('XDG_RUNTIME_DIR')
```

### Convert curl request to Ruby

See https://jhawthorn.github.io/curl-to-ruby.
