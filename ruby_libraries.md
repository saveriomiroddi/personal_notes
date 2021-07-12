# Ruby libraries

- [Ruby libraries](#ruby-libraries)
  - [APIs/Stdlib](#apisstdlib)
    - [URL/HTML encoding](#urlhtml-encoding)
    - [Strings](#strings)
      - [Encoding](#encoding)
    - [Date/time](#datetime)
      - [Date/time templated parsing](#datetime-templated-parsing)
    - [Maths/Rational](#mathsrational)
    - [CSV](#csv)
    - [JSON](#json)
    - [XML](#xml)
      - [REXML (`rexml/document`)](#rexml-rexmldocument)
      - [XPath](#xpath)
      - [Builder](#builder)
    - [YAML/Psych](#yamlpsych)
    - [OpenStruct [ostruct]/Struct](#openstruct-ostructstruct)
    - [Optparse](#optparse)
    - [open-uri](#open-uri)
    - [File/Dir/FileUtils/Pathname](#filedirfileutilspathname)
    - [Tempfile, Tmpdir](#tempfile-tmpdir)
    - [I/O and terminal](#io-and-terminal)
    - [Encryption (openssl)](#encryption-openssl)
    - [SecureRandom](#securerandom)
    - [Flock](#flock)
    - [Etc](#etc)
    - [StringIO](#stringio)
    - [Profiling (ruby-prof)](#profiling-ruby-prof)
    - [Gem](#gem)
      - [Version comparisons](#version-comparisons)
    - [ERB templates](#erb-templates)
    - [GC (garbage collection)](#gc-garbage-collection)
    - [Networking](#networking)
      - [HTTP](#http)
      - [Telnet](#telnet)
      - [SMTP](#smtp)
  - [Databases](#databases)
    - [SQLite 3](#sqlite-3)

## APIs/Stdlib

### URL/HTML encoding

Rigorously encoding is not fully addressed by the Ruby STL; see https://stackoverflow.com/a/13059657.

Conclusions:

- Use `CGI::escape` if you only need form escape;
- If you need to work with URIs, use [`Addressable`](https://rubygems.org/gems/addressable), it offers URL encoding, form encoding and normalizes URLs.

Ruby examples:

```ruby
CGI::escape("&&&")                            # URL-encode a string: "%26%26%26"
URI.encode_www_form(p1: "&&&", "p2" => "!!!") # URL-encode params: "p1=%26%26%26&p2=%21%21%21"

CGI::escapeHTML('"html"')       # escape HTML

CGI.unescapeHTML("html")        # decode HTML entities; use only for basic cases, as it' not 100% complete (gem: https://github.com/threedaymonk/htmlentities)
HTMLEntities.new.decode("html") # htmlentities gem
```

Addressable examples:

```ruby
Addressable::URI.form_encode(p1: "&&&", "p2" => "!!!") # "p1=%26%26%26&p2=%21%21%21"

# Other APIs are provided for unencoding, parsing etc. (pubmeds `Addressable::URI`)
```

### Strings

Useful methods:

- `casecmp(str)`: case insensitive comparison
- `center`, `ljust(int)`, `rjust(int)`
- `strip`, `lstrip`, `rstrip`
- `index(substr)`, `rindex(substr)`
- `slice(start[, end])`, `slice!`
- `start_with?`
- `lcomp`: doesn't exist; use `str.gsub(/^(Regexp.escape(expr))+/)`
- `% *values`: equal to `sprintf(str, *values)`

Splitting:

```ruby
# WATCH OUT!!!:
# - the default separator is /\s+/, which is convenient, but not intuitive
# - by default (`limit: nil`), empty fields are not included in the result; in order to include them,
#   specify `limit: -1`
split(separator[, limit])

split(/(separator)/)                  # retain the separator as separate token
split(/(?<=\+)\n(?=\+)/)              # retain the separator in the splitted fields via LB/LA (complex case: `\n` is discarded)

"ä\x00ß\x00\x00".split("\x00")        # ["ä", "ß"] - empty tokens are not included by default!!
"ä\x00ß\x00\x00".split("\x00", -1)    # ["ä", "ß", "", ""] - pass `-1` as limit in order to keep them
```

Substituting/matching:

```ruby
# The closure is not supported for [g]sub!.
#
'foobar'.gsub(/(ba.)/, '\1\1')                        # 'foobarbar'; WATCH OUT!: the replacement string can't be manipulated, e.g. `upcase()`
'foobar'.gsub(/(b)(a)/) { |match| $2.upcase + $1 }    # 'fooaBr'; `match` is a String
"Saverio <a@b.c>".match(/<(.*)>/)                     # returns MatchData object; [0] = entire match; [1..] = matching groups
str[/regex/, idx]                                     # same as `str.match(/regex/)[idx]`
"saverio".scan(/(ver(io))/)                           # [["verio", "io"]]
"saverio".scan(/cusumano/)                            # []

# when a non-capturing group ("?:") is used, gsub nonetheless replaces the entire
# string; this is also valid when using a block - in this case, the entire string will be
# passed as parameter.
# this is inconsistent with :scan, which will only extract the capturing groups.
#
# when using a block, in order to reference capturing groups, one can use the last match
# global variables:
#
"y:33".gsub(/(?:y:)(\d+)/) { "y:#{ $1.to_i + 1 }" }        # returns "y:34"

# in order to replace non-capturing groups with gsub, use the lookahead/behind, instead
# of non-capturing groups:
#
"abc".gsub(/a(?=b)/, 'x')                                    # returns 'xbc'

# Linux tool `tr`
#
"abcZHNC".tr("ZHNC", "*")                               # return "abc****"
```

#### Encoding

Also see [Data type Conversions](#data-type-conversions).

```ruby
String.new(str, encoding: enc)    # !! Defaults to ASCII-8 encoding !!
str.encode(encoding)
str.force_encoding(encoding)
str.valid_encoding?

# Clean non UTF-8 chars.
#
str.encode('UTF-8', invalid: :replace, undef: :replace, replace: '')

# scrub 4-bytes characters from an utf8 string (essentially, mysql's `utf8mb4` to `utf8mb3`)
#
string.each_char.select { |char| char.bytesize < 4 }.join

# Scripting encoding magic comment; must be in first or second line.
#
# [en]coding: UTF-8

# Scripting encoding.
#
__ENCODING__

# The external encoding is the encoding used when creating the IO object.
# If the internal enc. is specified, IO objects (valid for read and write) are transcoded
# to it before being read/write.
#
Encoding.default_(internal|external)

# File opening options
#
open(file, "r:<external_enc>:<internal_enc>")
open(file, "w:<external_enc>"

# ignore Byte Order Mark
#
IO.read(filename, 'bom|utf-8')
```

### Date/time

Operations/conversions:

```ruby
@time + seconds

@date >> @months; @date << @months                        # Adds/subtracts months to a date (!!)
@date.next_month(@months=1), @date.prev_month(@months=1)  # More readable; also supports `day`, `year`

@date_time.to_time.to_i                                   # DateTime to Unix time
```

There is no simple way to subtract an year. leap day should be taken into account, and even when considering this, is an year 365 or 366 days? If an year less is the same date but on the previous yes, a simple solution is to print and reparse the timestamp; alternatively, the more complicated way can be achieved using `Date.new(datetime.year - 1, 1, 1).leap?`, which is not suggested because it's very easy to make a mistake.

In order to parse a freeform time, use DateTime (required `time`), but mind the offset:

```ruby
# If the offset is not specified, it's assumed to be UTC.
#
DateTime.parse("10:40 #{Time.now.strftime("%z")}")     # => #<DateTime: 2020-10-27T10:40:00+01:00 ...>
DateTime.parse("thu 10:33 #{Time.now.strftime("%z")}") # => #<DateTime: 2020-10-29T10:33:00+01:00 ...>
```

#### Date/time templated parsing

- `%a`: The abbreviated weekday name ('Sun')
- `%A`: The full weekday name ('Sunday')
- `%b`: The abbreviated month name ('Jan')
- `%B`: The full month name ('January')
- `%c`: The preferred local date and time representation
- `%d`: Day of the month (01..31)
- `%H`: Hour of the day, 24-hour clock (00..23)
- `%I`: Hour of the day, 12-hour clock (01..12)
- `%j`: Day of the year (001..366)
- `%L`: Millisecond of the second (000..999)
- `%N`: Fractional seconds digits, default is 9 digits (nanosecond)
  - `%3N`: millisecond (3 digits)
  - `%6N`: microsecond (6 digits)
  - `%9N`: nanosecond (9 digits)
  - `%12N`: picosecond (12 digits)
- `%m`: Month of the year (01..12)
- `%M`: Minute of the hour (00..59)
- `%p`: Meridian indicator ('AM' or 'PM')
- `%S`: Second of the minute (00..60)
- `%U`: Week number of the current year,starting with the first Sunday as the first day of the first week (00..53)
- `%W`: Week number of the current year, starting with the first Monday as the first day of the first week (00..53)
- `%w`: Day of the week (Sunday is 0, 0..6)
- `%x`: Preferred representation for the date alone, no time
- `%X`: Preferred representation for the time alone, no date
- `%y`: Year without a century (00..99)
- `%Y`: Year with century
- `%z`: Time zone as hour and minute offset from UTC (+0900)
  - `%:z`: hour and minute offset from UTC with a colon (+09:00)
  - `%::z`: hour, minute and second offset from UTC (+09:00:00)
- `%Z`: Time zone name (CEST)
- `%%`: Literal ``%'' character

Combinations:

- `%F`: The ISO 8601 date format (%Y-%m-%d)
- `%R`: 24-hour time (%H:%M)
- `%T`: 24-hour full time (%H:%M:%S)

Examples:

```ruby
'%F %T'             # 2008-12-31 01:23:45
'%a %b %d %T %Z %Y' # Wed May 02 01:23:45 GMT 2014
'%m/%d/%y %I:%M %p' # 10/31/09 01:23 PM
'%YT%T%:z'          # 2014T14:20:54+00:00 (json format)
```

### Maths/Rational

```rb
r = Rational(8, 9)      # no new()!!
(r * 9).to_i == 8       # arithmetic yiels other Rational instances
x.gcd(y)                # Greatest commond divisor (Massimo Comun Divisore, MCD)

x = Math.log2(y)
x = Math.sqrt(y)        # square root; DO NOT USE `y ** 0.5`!!!
x = Math.sin(y_rad)     # remember that rad = deg / 180 * Pi
x = Math.cos(y_rad)
x = Math.pow(y[, m])    # if m is specified, it's performed `x^y % m`, without huge intermediate numbers
Math::PI
```

### CSV

Options:

- `headers: :first_row`
- `force_quotes: true`

```ruby
CSV.read(filename)                                        # Read csv; the output is a CSV::Table, which can be iterated.
CSV.read(filename).to_a                                   # Doesn't work as expected: first entry is the header; each entry is a flat array, not a hash.
CSV.parse(data_string[,options])                          # Parse a string
CSV.foreach(filename, options) { |row| ... }              # Stream (for large files)

csv_content = CSV.generate { |csv| csv << [values] }      # Write to string
CSV.open("/path/file.csv", "w", options) { |csv| ... }    # Write to file
CSV.open('links.csv', 'ab', options) { |csv| ... }        # Append to file
```

Row can be indexed as a hash; empty fields are parsed as nil.

Utils:

```ruby
CSV.read(csv_file)[0]                                     # Read headers. There is no obvious way from the API (what if the file has only headers?)
row.to_hash.merge(row) { |_, value| value.to_s }          # Simple way to convert each row nil values to blank strings
```

### JSON

```ruby
JSON.parse(string)                                        # Returns the tree; the root object class depends on the input (eg. Array, Hash, ...)
string = JSON.generate(input)                             # `input` can be a hash
string = JSON.pretty_generate(input)
```

### XML

#### REXML (`rexml/document`)

```ruby
xml_str = "\
<myroot>
  <internal_1>
    <leaf_1/>
    <leaf_2>33</leaf_2>
  </internal_1>
  <internal_2>1</internal_2>
  <internal_2>2</internal_2>
</myroot>"

root = REXML::Document.new(xml_str)

# Watch out! REXML formatters insert newlines, without any option, in the text values,
# so if newlines are meaningful, REXML must not be used – use nokogiri.
#
xml_formatter = REXML::Formatters::Pretty.new  # pretty print
xml_formatter.compact = true
xml_formatter.write(root, $stdout)

myroot.elements.[reverse_]each { |child| }      # iterate the child elements
myroot.elements[1..-1]                          # !!! NOT supported !!!
myroot.elements['internal_2']                   # pick the first child named element
myroot.elements.to_a('//myroot/internal_2')     # access via XPath

internal_2.parent   # (Element myroot)
internal_2.name     # 'internal_2'
internal_2.text     # '2'

internal_3 = myroot.add_element("internal_3", 'attr' => 'value')  # add element, optionally with attributes
myroot.delete(internal_3)                                         # remove element
internal_3.parent = nil                                           # remove element (other way)

internal_1.attributes['type'] = 'array'         # add/remove attributes
internal_1.attributes.delete('type')

internal_1.insert_before( 'leaf_2', REXML::Element.new('leaf_between'))
internal_1.insert_after( 'leaf_2', REXML::Element.new('leaf_end'))

internal_4 = myroot.add_element("internal_4", 'type' => "mytype")
internal_4.text = 'mytext'
```

#### XPath

```ruby
# Search by multiple attributes:
#
index_document.elements.to_a( %Q|//RDF:RDF/RDF:Description[@NS1:type="folder" and @title="title"]| )

# Search relative to element:
#
folder_content_element.elements.to_a( 'RDF:li' )
```

#### Builder

- There is no need of a target; each action on it returns the string. but NEVER use `to_s`!!!
- Every block has the xml element as argument, but the 'xml' variable can be used as well!

```ruby
# Base example

xml = Builder::XmlMarkup.new [:target => some_string, :indent => spaces]
xml.instruct!
xml.pizzas do # here we aren't using a named element pointer
   xml.4_stagioni :type => 'buona' do                                          # !!! REQUIRES do/end, NOT {} !!!
     xml.ingrediente :type => 'scaduto', 'catrame'                             # :type = attribute
     xml.photo do |photo_xml|                                                  # here we are using a named element pointer
       photo_xml.cdata! 'my cdata!'
     end
   end
end
def generate_xml_children(xml, level, &block) # recursion works!
  node_name = [nil, :shows, :events][level]
  xml.__send__( node_name, :type => 'array', &block)                         # use __send__, because send method is interpreted as xml element
end
```

### YAML/Psych

The `yaml` library is actually an alias for `psych`. Comments are always ignored when parsing!

```ruby
YAML.load_file(filename)  # Parse from file
YAML.load(string)         # ... from string

YAML.dump(object[, io])   # Convert to string, optionally into IO
object.to_yaml            # ... convenience
```

### OpenStruct [ostruct]/Struct

Struct is in the stdlib; it doesn't accept new arguments.

```ruby
os = OpenStruct.new can also pass a hash!
os.name = "John"
os['surname'] = "Smith" # alternate assignment form
puts os.name, os.surname

# !!! Substruct.new(a: 1).a => 1 !!!
# This works because Struct.new returns a class (!); if not subclasses, it's accessed as Struct.new(...).new
#
class SubStruct < Struct.new(:a, :b); end

# Struct Form with explicit class naming.
# It seems that this doesn't accept scoping (= forces the Struct module)
#
klazz = Struct.new('MyStruct', :a, :b) do
  def mymethod; end
end
klazz.new('aval', 'bval')
Struct::MyStruct

klazz.members # returns the defined members ([:a, :b], in this case)

# Struct with implicit name modules.
# Struct::MyStruct won't exist, in this case.
#
module MyModule
  MyStruct = Struct.new(:a, :b)
end

MyModule::MyStruct.new('aval', 'bval')
```

### Optparse

```ruby
# Set default options
options = { garbage: false, logfile: nil, volume: 1.0, is_moron: nil, crappy: true }

optparse = OptionParser.new do |op|
  op.banner = "Usage: #{File.basename(__FILE__)} [-b|--base] [-p PARAM] [-c PARAM] [-e PARAM] [-[no-]n|--[no-]negated]"

  # Base form; if there is more than one param (e.g. the long version), the last array entry is the
  # description.
  #
  op.on('-b', '--base', 'Base form') do
    options[:base] = true
  end

  # With optional parameter (not enforced, due to default); upper case is not necessary.
  #
  op.on('-p PARAM' 'Optional param') do |param|
    options[:param] = param || 'default_value'
  end

  # Class conversion. An Array can be passed, it takes the form 'a,b,c'
  #
  op.on('-c PARAM', Float, 'Converted param') do |param|
    options[:converted] = param
  end

  # Enum params. Can be specified as symbol or string, and are passed the same
  #
  op.on('-e PARAM', ['yes', 'no', 'maybe']) do |param|
    options[:enum] = param
  end

  # Negation; passes false when negated, true otherwise.
  # When specified in long form, it's automatically activated for the short one (in this example,
  # '-no-c').
  #
  op.on('-n', '--[no-]negated') do |param|
    options[:negated] = param
  end
end

# Removes options/params from ARGV
#
optparse.parse!

puts "Options: [#{ options.inspect }]"
```

### open-uri

Open files/download http content; automatically detects the appropriate class for loading the stream content:

```ruby
open('http://cippa-lippa').read
```

### File/Dir/FileUtils/Pathname

```ruby
File.dirname(path)                              # Parent dir
File.extname(fname)                             # Extract the extension (including leading dot); if there are multiple dots, only the last
                                                # one is extracted

Dir.pwd                                         # working path
Dir.chdir(path) { }                             # Change current dir; WATCH OUT! not thread safe! see https://bugs.ruby-lang.org/issues/9785
Dir.home(user)                                  # Find home of the given user

Dir.foreach(dosPath){|fname|}                   # not recursive; includes '.[.]'
Dir.glob(antPath){|fname|}                      # glob format; 'path/**/pattern' => '**' recurses under path
Dir["**/*"]                                     # recursive dir list; can prepend dir; dirs before files

Dir.mkdir                                       # not recursive
FileUtils.mkdir_p(path)                         # recursive

Dir.delete(dosPath)                             # not recursive
FileUtils.remove_dir(path, true)                # recursive

FileUtils.touch(filename)
FileUtils.chmod(permissions, filename)          # change permissions; can be symbolic, e.g. `+x`
FileUtils.chown(user[, group[, filename]])      # change owner - File.chown needs the group/user id!!

FileUtils.cp(src, dest, **options)              # copy file
```

Paths handling:

```ruby
File.realpath("/path/to/symlink")               # get the real filename from symlinks:

require 'pathname'; Pathname.new(@path2).relative_path_from(Pathname.new(@path1)).to_s # find the directory relative to another directory
```

File opening/closing (flags):

```ruby
# If a file needs to be closed deterministically, then do it manually - GC collection doesn't guarantee
# that it will be collected/closed.
#
f = File.open(@file, File::CREAT | File::WRONLY | File::APPEND) { } # append to file, creating if non existent
f.close
```

File locking, via flock:

```ruby
# WATCH OUT!! This strategy doesn't play well with forked processes, if the file handle is acquired
# before forking; in this case, the lock will be acquired on both processes (!).

lock_file = File.open('/tmp/mylock', File::RDWR | File::CREAT)

# Exclusive lock, non-blocking
if lock_file.flock(File::LOCK_EX | File::LOCK_NB)
  # ... critical area ...
  # Make sure the handle is not GC'ed!
  $lock_file = lock_file
else
  # See note in "File opening/closing"
  lock_file.close
end
```

### Tempfile, Tmpdir

Do not use tempfile for references that stick around!! Tempfile instances are always deleted on garbage collection.

```ruby
file = Tempfile.new('prefix')
filename = file.path
file.close
File.unlink(file)  # optional

# <naming> can be either "prefix" or ["prefix", ".extension"]
file = Tempfile.new(<naming>)

Tempfile.open(<naming>) { |f| operation(f) }    # Creates, operates, but doesn't immediately delete
Tempfile.create(<naming>)                       # Opens and closes, but doesn't immediately delete
Tempfile.create(<naming>) { |f| operation(f) }  # Creates, operates, and deletes

# Generates a temporary filename with `a`/`.png` prefix/extension, in the system temporary directory.
# !!This is the best API for tempfiles that need to stick around!! Note that no file is created.
# The block is ugly, but required.
#
require 'tmpdir'
Dir::Tmpname.create(['a', '.png']) { }

# Find system temporary directory
require 'tmpdir'
Dir.tmpdir
```

### I/O and terminal

```ruby
require 'io/console'; $stdin.getch              # Read a single char
```

### Encryption (openssl)

```ruby
cipher = OpenSSL::Cipher::Cipher.new(algo)
cipher.encrypt

iv = cipher.random_iv

cipher.key = key
cipher.iv  = iv

ciphertext = iv + cipher.update(plaintext) + cipher.final

ciphertext = Base64.encode64(ciphertext) if base_64_encoding

# decoding

cipher = OpenSSL::Cipher::Cipher.new(algo)
cipher.decrypt

cipher.key = key

cipher.iv = ciphertext.slice!(0, cipher.iv_len)
plaintext = cipher.update(ciphertext) + cipher.final

# useful

cipher.random_key          # generate a key
```

### SecureRandom

Interesting :crypt API; generate a linux password hash

```sh
ruby -rsecurerandom -e 'print "Password: "; puts gets.chomp.crypt("$6$" + SecureRandom.hex(8))'
```

### Flock

Flags:

- `LOCK_EX` - Exclusive
- `LOCK_SH` - Shared lock
- `LOCK_UN` - Unlock
- `LOCK_NB` - Non-blocking acquisition; combine using logical or. Returns `0` (success) or `false` (failure).

The lock is released on GC (/exit), so if one wants it to hang around, it must be assigned to global variable.

```ruby
# WATCH OUT: don't use `w` file mode. Default permissions: 0664.
#
$lock_file = File.open("/tmp/coverband-flock", File::RDWR | File::CREAT)
$lock_file.flock(File::LOCK_EX | File::LOCK_NB)
```

### Etc

```ruby
Etc.getlogin                                # Current user
Etc.getpwuid.dir                                # Get current user home dir. For generic users, use `Dir.home`
```

### StringIO

```ruby
# WATCH OUT! If the `a`ppend mode is not specified, the string passed is overwritten!
#
StringIO.new("start_string", "a")

# WATCH OUT! :puts adds a newline, but chomps the parameter's last trailing newline
#
buffer.print "line\n"    # one trailing newline
buffer.puts  "line\n"    # one trailing newline
buffer.puts  "line\n\n"  # two trailing newlines
buffer.puts  "line", ""  # two trailing newlines
```

### Profiling (ruby-prof)

NOTE: Garbage collection should always be stopped before profiling! (Currently) GC pauses can't be detected by the profiler.

```ruby
result = RubyProf.profile do
  # ...
end

# Different outputs (with openers)
#
RubyProf::GraphHtmlPrinter.new(result).print(File.open('/tmp/prof.htm', 'w'), min_percent: 5); `xdg-open /tmp/prof.htm`
RubyProf::CallStackPrinter.new(result).print(File.open('/tmp/prof.htm', 'w')); `xdg-open /tmp/prof.htm`
RubyProf::CallTreePrinter.new(result).print(path: '/tmp', profile: 'prof'); `qcachegrind /tmp/prof.callgrind*`
```

### Gem

#### Version comparisons

Rigorous way to compare versions:

```ruby
'3.0' > '10.0'                                     # true (wrong!)
Gem::Version.new('3.0') > Gem::Version.new('10.0') # correct
Gem.ruby_version > Gem::Version.new('10.0')        # ready API to retrieve the ruby version
```

### ERB templates

Simple rendering logic example; keep in mind the binding considerations.

```ruby
template_content = IO.read(template_file)

# WATCH OUT! Instance variables of the binding object are different from instance variables in the
# binding - ie. using :instance_variable_set will not work as expected.
# There are different approaches. This is st00pid simple; for a more sophisticated one, see
# https://stackoverflow.com/a/41590728.
#
variables.each { |name, value| TOPLEVEL_BINDING.eval "@#{name} = #{value.inspect}" }

# It's possible to render with a local context, and evaluate in the global one, however, it doesn't
# have any particular advantage (at least, for the use case).
#
rendered_template = ERB.new(template_content).result(TOPLEVEL_BINDING)

TOPLEVEL_BINDING.eval(rendered_template, template_file)

# Since we're working on the top level binding (so that classes will be in the global namespace),
# it's better to clean up :-)
#
variables.keys.each { |name| TOPLEVEL_BINDING.eval "remove_instance_variable '@#{name}'" }
```

### GC (garbage collection)

APIs:

- `enable`/`disable`
- `start`

WATCH OUT! When profiling, the GC should be disabled, because profilers may not be able to recognize GC pauses.


### Networking

#### HTTP

```ruby
# An URI instance ("URI(address)") can be used, whose :host, :port, :request_uri can be used
#
Net::HTTP.start(uri.host, use_ssl: true) do |http|
  request = Net::HTTP::Get.new(uri)   # for post: Post.new(uri) + body=data
  request.basic_auth @user, @pwd
  request['Accept'] = 'application/vnd.github.v3+json'
  response = http.request(request)
  status_header = response['Status']
  headers = response.to_hash
end
```

#### Telnet

```ruby
# Set `telnetmode` to false in order to connect eg.to pop3.
#
Net::Telnet.new('host' => addr, 'port' => port, 'telnetmode' => false)

# Connecting to memcached (or others).
# The 'Prompt' is essentially what is returned after a command is executed.
#
connection = Net::Telnet.new( 'Host' => host, 'Port' => 11211, 'Prompt' => /^OK\n/ )
connection.cmd( 'flush_all' )
```

#### SMTP

https://stackoverflow.com/a/3559598

```ruby
# Pay much attention to the newlines, which are fundamental.
#
sender_email, recipient_email    = 'sender@email.com', 'recipient@email.com'
sender, recipient, subject, body = "Sender <email>", "Recipient <email>", "Subject", "Body"
marker                           = "MARKER_=+!£$%^&*()-#'~@`¬|,.;/][:?}{_ENDMARKER"

raise "The body includes the marker!" if body.include?(marker)

mail_headers = %Q{\
From: #{sender}
To: #{recipient}
Subject: #{subject}
}

if attachments_data.size > 0
  mail_headers << %Q{\
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="#{marker}"
--#{marker}
Content-Type: text/plain
Content-Transfer-Encoding: 7bit

}
  attachment_section = attachments_data.inject("") do |attachments_buffer, (filename, content) |
    encoded_content = [content].pack('m')  # base 64

    attachments_buffer += %Q{
--#{marker}
Content-Type: application/octet-stream; name=\"#{filename}\"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="#{filename}"

#{encoded_content}\
}
  end

  attachment_section << "--#{marker}"
end

mail_text = "#{mail_headers}#{body}#{attachment_section}--\n"

# Basic send.
#
Net::SMTP.start('localhost') do |smtp|
   smtp.send_message(mail_text, sender_email, [recipient_email])
end

# TLS send (see https://stackoverflow.com/a/3559598).
#
smtp = Net::SMTP.new 'smtp.gmail.com', 587
smtp.enable_starttls
smtp.start("domain", "user", "pwd", :login) do
  smtp.send_message(mail_text, sender_email, [recipient_email])
end
```

## Databases

### SQLite 3

```ruby
require 'arrayfields' # makes rows accessible also as hash in addition to be arrays

db = SQLite3::Database.new("test.db")

rows           = db.execute(query[, params])
columns, *rows = db.execute2(query)

db.execute(query) { |row| ... }

row   = db.get_first_row("select * from table")
count = db.get_first_value("select count(*) from table")

db.execute("insert into foo (?)", SQLite3::Blob.new("\0\1\2\3\4\5"))  # insert blob
db.execute_batch(queries)  # accepts multiple statements (normal #execute doesn't); no blocks accepted.

db.results_as_hash  = true
db.type_translation = true  # translate the data to the declared data type

database.transaction { |db| db.execute(...) }
db.transaction; db.execute(...); db.commit

db.execute "SELECT name FROM mysql_master WHERE type = 'table'"  # get table names

db.extend(SQLite::Pragmas); db.table_info(table_name)  # table metadata
```