# Rust
- [Rust](#rust)
  - [Cargo](#cargo)
  - [Syntax/basics](#syntaxbasics)
    - [Basic structure/Printing/Input](#basic-structureprintinginput)
      - [Printing](#printing)
    - [Variables/Data types](#variablesdata-types)
    - [Basic operators/operations](#basic-operatorsoperations)
    - [Closures](#closures)
    - [Ranges and `std::iter::Iterator` methods](#ranges-and-stditeriterator-methods)
    - [Arrays/Vectors/Slices](#arraysvectorsslices)
    - [Hash maps](#hash-maps)
    - [Strings](#strings)
      - [Internal representation (bytes/chars/graphemes)](#internal-representation-bytescharsgraphemes)
    - [For/while (/loop) loops](#forwhile-loop-loops)
    - [If/then/else](#ifthenelse)
    - [Enums](#enums)
    - [Option<T>/Result<T, Error>](#optiontresultt-error)
    - [Pattern matching](#pattern-matching)
      - [Error handling](#error-handling)
    - [Structs](#structs)
    - [[Static] Methods](#static-methods)
  - [Ownership](#ownership)
    - [Move](#move)
    - [Borrowing](#borrowing)
    - [Dangling pointers](#dangling-pointers)
    - [Lifetimes](#lifetimes)
    - [Slices](#slices)
  - [APIs/Crates](#apiscrates)
    - [Files/streams handling](#filesstreams-handling)
    - [Testing](#testing)
    - [String/char-related](#stringchar-related)
    - [Random (`rand`)](#random-rand)
    - [Regular expressions (`regex`)](#regular-expressions-regex)
    - [Date/times (standard)](#datetimes-standard)
    - [Date/times (`chrono`)](#datetimes-chrono)
    - [Commandline parsing (`clap`)](#commandline-parsing-clap)

## Cargo

Base operations:

```sh
cargo new "$project_name"
cargo check                          # check for errors
cargo run
cargo build [--release]              # builds (default: debug); if necessary, updates the crates index, and installs the dependencies
cargo update                         # updates the crates index, and the installed dependencies
cargo test
cargo fmt
cargo clippy                         # linter
cargo doc [--open]                   # builds and optionally opens docs for the installed crates
```

Configuration file example:

```toml
# Enable nightly features; `strip` is on 1.46
cargo-features = ["strip"]

[package]
name = "rust"
version = "0.1.0"
authors = ["Saverio Miroddi <saverio.etc@etc.com>"]
edition = "2018"

# Customize the binary target.
[[bin]]
name = "play"
path = "src/play.rs"

[dependencies]
rand = "0.7.3"

[profile.release]
strip = "symbols"
```

Workspace: manage multiple projects.
Using cargo from root requires the member name; otherwise, each member can be treated as an individual project.

```toml
# Some settings must be in the workspace configuration when using a workspace, eg. nightly features.

[workspace]
members = ["playground", "rust_programming_by_example"]
```

Versioning is pessimistic by default.

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.

## Syntax/basics

### Basic structure/Printing/Input

```rust
use std::io;
use std::io::Write; // bring flush() into scope

// "attributes": metadata with different purposes.
// #![...] is at crate level.
//
#[allow(dead_code)]
fn testing(n: u32) -> String {
  if n > 10 {
    panic!("Error message!");
  }
  // In order to return a value without using `return`, omit the semicolon.
  //
  String::from("abc")
}

fn main() {
  print!("Enter guess: ");
  io::stdout().flush().unwrap(); // makes sure that the output is flushed, since O/S generally do it per-line.

  // `mut`: mutable.
  // the `new` function is not dictated by the language, but a common practice.
  // static methods are called "associated functions".
  //
  let mut guess = String::new();

  // `&`: reference
  // `&mut` is necessary, if expected, even if the variable is mutable.
  // `read_line()` returns the enum `io::Result`; the "variants" are `Ok` and `Err`.
  //
  io::stdin().read_line(&mut guess).expect("Failed to read guess!");

  // Placeholder: `{}`
  //
  println!("Guess: {}", guess);
}
```

#### Printing

```rust
println!("{:#?}", vec);                 // generic pretty printing
println!("{:?}", vec);                  // `Debug` format (requires the `Debug` trait)

eprintln!("Error!");                    // print on stderr!

println!("{:.2}", f);                   // formatted printing (rounds float)
```

### Variables/Data types

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required

((1u128 << CONST_U64) - 1) as u64    // WATCH OUT the priorities! In this example, the brackets are all required!
```

Integer types:

- `[iu](8|16|32|64|128|size)`
  - the `size` ones depend on the architecture
  - the default, if not specified and can't be inferred, is `i32`

Max value function (example): `u64::max_value()`

Floats types: `f(32|64)`.

Integer literals:

```rust
0xff      // hex
0o77      // octal
0b1100    // binary
b'A'      // byte; only `u8`; require single quotes
```

All number literals support the type as suffix (e.g. `32u8`), and the (cosmetic) underscore.

Extra string literals:

```rust
b"hello"                // byte (ASCII) string
r"hello"; r#"hello"#    // raw string (doesn't process escapes)
br"hello"; br#"hello"#  // byte raw string
```

Char literals:

```rust
'🤯'          // 4 bytes, require single quotes
```

Escapes ([Tokens](https://doc.rust-lang.org/reference/tokens.html) subset):

```rust
"\u{0304}"    // 24-bit Unicode character code (up to 6 digits); valid also as char
```

Tuples:

```rust
let foo = ("bar", "baz");
let (mut bar, mut baz) = foo;     // multiple assignment (unpacking); foo can also be a tuple literal
let first_element = tuple.0;      // tuple indexing

// Use a tuple as function argument
fn are(dimensions: (u32, u32)) -> u32 {
  dimensions.0 * dimensions.1
}
```

For strings, see the [Strings chapter](#strings).

### Basic operators/operations

```rust
val += 1; val -= 1;             // increment/decrement value (no postfix)
std::mem::swap(&mut a, &mut b); // !! swap two variables !!

let val = 10_u64.pow(2);        // exponentiation (power)
(f * 100.0).round() / 100.0;    // ugly: round to specific number of decimals (also see #printing)
```

### Closures

Equivalent of Ruby blocks!

```rust
let multiple_of_10 = |x| x % 10 == 0; // yay!
(0..100).any(multiple_of_10);         // double yay!
```

### Ranges and `std::iter::Iterator` methods

General form: `[start] .. [[=]end]`.

Ranges are:

- lazy;
- open ended on the `end`, unless `=` is specified.

`std::iter::Iterator` methods, implemented by Range:

```rust
map(|x| x * 2)               // Ruby map!!! 😍😍😍
fold(a, |a, x| a + x)        // Ruby inject!!! 😍😍😍
filter(|x| x % 2 == 0)       // Ruby select
find(|x| x % 2 == 0)         // find first element matching the condition
rev()                        // reverse. WATCH OUT, UNINTUITIVE: since it's not inclusive, it goes from 99 to 0.
any(|x| x == 33)             // terminates on the first true
all(|x| x % 2 == 0)          // terminates on the first false
nth(n)                       // nth element (0-based)
take(n)                      // iterator for the first n elements
enumerate()                  // iterator (index, &value)
join("str")                  // join using str
zip(iter)                    // zip two arrays (iterators)!!!
sum::<T>()

chunks(n)                    // iterate in chunks of n elements; includes last chunk, if smaller
chunks_exact(n)              // iterate in chunks of n elements; does not include the last chunk, if smaller
windows(n)                   // like chunks, but with overlapping slices

// transform an iterator into a collection
collect()
collect::<Vec<i32>>()
collect::<Vec<_>>()
```

### Arrays/Vectors/Slices

Arrays (immutable, so they're allocated on the stack):

```rust
let my_list = [1, 2, 3];
let my_list = [true; 4];                // 4 elements initialized as true; won't work with variable size (use a Vec)
let my_list: [u32; 3] = [1, 2, 3];      // with data type annotation; ugly!
let mut my_list: [Option<u32>; 3] = [None; 3];  // with Option<T>; super-ugly!
```

Vectors (mutable):

```rust
let mut vec = Vec::new();               // Basic (untyped) instantiation (if the type can be inferred)
let mut vec: Vec<i32> = Vec::new();     // Basic, if type can't be inferred
let mut vec = vec![1, 2, 3];            // Macro to initialize a vector from a literal list
let mut vec = vec![true; n];            // Same, with variable-specified length and initialization

vec[0] = 2;
vec.get(2);                             // Option version
vec.push(1);                            // Push at the end

let val = &vec[0];
vec.pop();                              // Pop from the end

vec.len();

// Pattern matching!
match v.get(2) {
  Some(entry) => println!("The third element is {}", entry),
  None => println!("There is no third element."),
}

vec.iter();                             // iterator
vec.extend([1, 2, 3].iter().copied());  // append a list
vec.extend(&[1, 2, 3]);                 // borrowing version

vec.first();
vec.last();
```

Arrays implement the `Debug` trait.

Using an Enum to store different data types in an array!!:

```rust
enum CsvRow {
  Int(i32),
  Float(f64),
};

let mut vec = vec![];

vec.push(CsvRow::Int(1));
vec.push(CsvRow::Float(2.0));
```

Slices apply to arrays as well:

```rust
let array = [8u32; 5];
let slice = &array[..];         // type is `&[u32]`
```

See the [ownership chapter](#ownership), for the related properties.

### Hash maps

The default hashing function is cryptographically secure!!. For faster versions, must use a crate.

```rust
use std::collections::HashMap;

let mut map = HashMap::new();

map.insert("b", 10);
map.insert("b", 10);            // Overwrites the existing value

// `entry()` gets the value for in-place modification.
// `or_insert()` sets the given value if the key doesn't exist; its return value can be used to
// modify the value in-place.
//
let entry = map.entry("b").or_insert(50);
*entry = 100;

// Getters use references.
//
map["b"];
map.get("a");                   // Option

// Invalid! Once a key is inserted, it's owned by the hash map!
//
let key = String::from("abc");
map.insert(key, 20);
println!("{}", key)
```

Conveniences:

```rust
// Create a hashmap from multiple arrays.
//
let scores: HashMap<_, _> = teams
    .iter()
    .zip(initial_scores.iter())
    .collect();
```

### Strings

There are two types of strings:

- literals (`&'static str`); hardcoded in the executable.
- `std::string::String`s; allocated on the heap: `String::from("text")`.

Then, there are the slices (don't forget the `&` operator!!!).

```rust
// The `String` data type is mutable.
//
let string: String = String::from("pizza!");
let string: String = "pizza!".to_string();
let mut string = String::new();

let slice: &str = &string[0..3];
let literal: &str = "Your pizza";             // string literals are slices!
```

Using strings with functions:

```rust
// Valid, but forces to pass a string
//
fn str_method(s: &String) -> &str { s }

// Better: accepts slices
//
fn str_method(s: &str) -> &str { s }

// All valid!
//
str_method(&string[..]);
str_method(&literal[..]);
str_method(literal);
```

#### Internal representation (bytes/chars/graphemes)

A `String` is a wrapper over a `Vec<u8>`.

Rust has three notions of string composition. Example `नस्` (the second character is accented):

- bytes: `[224, 164, 168,   224, 164, 184,   224, 165, 141]`
- Unicode scalars (codepoints): `['न', 'स', ' ्']`
- grapheme clusters: `"न", "स्"`

Rust doesn't handle graphemes natively (requires a crate).

Accessing:

```rust
// Not valid: direct array indexing
//
"ननन"[0]

// Range access, but panics if the sequence returned is not at char boundary.
&"ननन"[0..3]

// Extract chars. WATCH OUT! Does not split into graphemes, e.g. "ü" will be 2 chars (!).
string.chars();

// Extract bytes.
string.bytes();
```

### For/while (/loop) loops

For loops iterate over ranges:

```rust
// For the reverse iteration, see rev() in the Ranges section, with warning(s).
//
for x in 0..10 { }

// Custom increment/decrement. See previous comment; goes from 98 to 0.
//
for x in (0..100).step_by(2).rev() {}

// Iterate an array/vector
//
for i in collection.iter() { }
for i in &collection { }
for i in &mut collection { *i *= 2 }
```

While:

```rust
while n > 0 {
  n -= 1;
};
```

Loop (infinite):

```rust
loop {
  if true {
    break;    // can return a value
  }
};
```

### If/then/else

```rust
if x > 5 {
  println!("{}!", x);
} else if x == 4 {
  println!("{}~", x);
} else {
  println!("{}", x);
}

// If with (multiple) assignment.
//
let (a, b) =
  if true {
    (1, 2)
  } else {
    (3, 4)
  };

// If let (see enum+pattern matching sections)
//
if let Coin::Quarter(state) = coin {
  println!("State quarter from {:?}!", state);
} else {
  count += 1;
}

// While let: same. In this case, with stacked Option<T>.
//
while let Some(Some(value)) = optional_values_vec.pop() {
  println!("current value: {}", value);
}
```

### Enums

Definition:

```rust
// Each entry is called "variant".
// The Debug trait is needed for printing.
//
#[derive(Debug)]
enum IpAddrKind {
  V4,
  V6,
}

// Enum with associated data; it can be of any kind, e.g. anonymous structs.
//
enum IpAddr {
  V4(u8, u8, u8, u8),
  V6(String),
}

// Methods can even be defined on enums!
//
impl IpAddr {
  fn is_local(&self) -> bool { true }
}
```

Assignment and invocation:

```rust
let four = IpAddrKind::V4;
let home = IpAddr::V4(127, 0, 0, 1);

fn route(ip_kind: IpAddrKind) { }
```

### Option<T>/Result<T, Error>

Foundation of Rust. In order to use the contained value, we must extract (and test) it.

```rust
enum Option<T> {
  Some(T),
  None,
}

// For None, the type must be specified, otherwise it can't be inferred.
//
let some_number = Some(5);
let absent_number: Option<i32> = None;

// Convenient syntax for decoding a Result and returning the Err, if any.
//
let value = method()?;

// Convenient pattern
//
method.unwrap_or_else( |err | {
  println!("Problem parsing arguments: {}", err);
  std::process::exit(1);
});
```

See next section for pattern matching.

### Pattern matching

```rust
// Generic match
//
let xxx = match val {
  1 | 2 => "1 or 2",
  _     => {
    "other"
  },
};

// Match enum; all the entries must be exhausted (or the `_` placeholder must be used).
// The second case is a variant with an associated value!
//
// See `if let` for an alternative structure.
//
match message {
  Message::Move{x, y} => { self.position = Point { x: x, y: y } },
  Message::Echo(s) => { self.echo(s) },
  Message::ChangeColor(c1, c2, c3) => { self.color = (c1, c2, c3) },
}

// Match Option<T>, and example usage.
//
fn plus_one(x: Option<i32>) -> Option<i32> {
  match x {
    None => None,
    Some(i) => Some(i + 1),
  }
}
let six = plus_one(Some(5));
let none = plus_one(None);

// Match Result<T, Err>
//
match Url::parse(&line) {
  Ok(url) => println!("ok"),
  _ => println!("ko"),
};
```

#### Error handling

Handle errors via enum matching:

```rust
let num: u32 = match num_string.parse() {
  Ok(num) => num,
  Err(_)  => panic!("Custom error!"),
};
```

### Structs

```rust
// Define a struct
struct User {
  username: String,
  active: bool,
}

// Instantiate it
let mut user = User {
  username: String::from("sav"),
}

// Update it. Requires the whole struct to be mutable!
user.active = true
```

Printing requires `Debug` trait!

Empty ("unit-like") struts can also be created, although they have a particular role.

Tuple structs:

```rust
struct Color(i32, i32, i32);
let black = Color(0, 0, 0);
```

tuple structs with same definition cannot share instances.

Odd functionality:

```rust
// "Field init shorthand" syntax for reducing struct definitions in functions.
fn create_user(email: String) -> User {
  User {
    email,
    active: true,
  }
}

// "Struct update" syntax to copy part of another instance.
let mut user2 = User {
  active: false,
  ..user
}
```

### [Static] Methods

Methods definition (essentially, struct functions)

```rust
struct Rectangle {
  width: u32,
  height: u32,
}

// Multiple methods can be defined in the same `impl` block (and multiple blocks can be defined).
impl Rectangle {
  // Methods can also borrow `self` mutably.
  // Methods taking ownership of `self` are rare and specific.
  fn area(&self) -> u32 {
    self.width * self.height
  }
}

impl Rectangle {
  fn square(size: u32) -> Rectangle {
    Rectangle { width: size, height: size }
  }
}
```

## Ownership

### Move

Variables can be simple (eg. integers) or complex (eg. strings); by default Rust copies the "simple" part (shallow copying).

When variables go out of scope, `drop` is invoked, freeing the memory:

```rust
{
  let val = String::from("text");
  // drop invoked!
}
```

On assignments/functions, simple data types are copied:

```rust
{
  let a = 5;
  let b = a;

  println!("{}", a); // valid
}
```

but complex ones are _moved_; ownership is transferred when assigning or passing to functions:

```rust
{
  let a = String::from("text");

  let b = a;

  // Invalid, unless the `Copy` trait is implemented, which declares that moved variables are still
  // usable. If the `Drop` trait is implemented, `Copy` is not allowed.
  //
  println!("{}", a);

  let c = my_function(b);

  // Invalid as well!
  //
  println!("{}", b);

  // If the function returns the variable, ownership is transferred.
  //
  // Valid!
  //
  println!("{}", c);
}
```

### Borrowing

Passing references doesn't transfer ownership; this is called _borrowing_:

```rust
{
  let a = String::from("text");

  my_function_ref(&a);

  // Valid!
  //
  println!("{}", a);
}
```

Borrowing mutable references has restrictions:

```rust
{
  let mut s = String::from("hello");

  {
    let r1 = &mut s;

    // No more than one mutable reference to the same value in the same scope:
    //
    // Invalid!
    //
    let r2 = &mut s;
  }

  {
    let r1 = &mut s;
  }

  // Different scope.
  //
  // Valid!
  //
  let r2 = &mut s;

  // Multiple immutable references can be in the same scope, but a mutable one excludes any other.

  let mut s2 = String::from("hello");
  let r1 = &s;

  // Valid!
  //
  let r2 = &s;

  // Invalid!
  //
  let r3 = &mut s;
}
```

### Dangling pointers

In Rust, it's not possible to have dangling pointers:

```rust
// Invalid!
//
fn dangling -> &String {
  let s = String::from("text");
  &s
}

// Must return the string directly.
//
// Valid!
//
fn not_dangling -> String {
  let s = String::from("text");
  s
}
```

### Lifetimes

Functions may need to know the lifetime of an object, in order to make sure that the resource is not freed prematurely.

For example, here, the returned vector is bound to the matches; if `matches` is freed, the content of the vector may be dangling!

```rust
fn extract_interval_arguments<'a>(matches: &'a clap::ArgMatches) -> Vec<&'a str> {
    matches
        .values_of("INTERVALS")
        .unwrap()
        .collect::<Vec<&str>>()
}
```

### Slices

Slices can refer to arrays and strings. They are immutable references, so the ownership needs to be considered:

```rust
{
  let mut s = String::from("margherita");

  let slice = &s[..]; // Immutable borrow
  s.clear();          // Mutable borrow: Invalid!

  println!("{}", slice);
}
```

String slices are at *byte* points!

Using string slices as arguments is preferrable to string references, as they're more generic (they can also take strings).

## APIs/Crates

### Files/streams handling

```rust
std::fs::read_to_string(filename) -> Result<String, Error>; // content must be valid UTF-8; filename can be relative.

// Buffered read; requires the import below
//
use std::io::prelude::*;
//
// Read line by line
//
let f = File::open("log.txt")?;
let mut reader = BufReader::new(f);
let mut line = String::new();
let len = reader.read_line(&mut line)?;

// Read whole lines (two ways: iteration, vector)
for line in reader.lines() { println!("{}", line?); }
let lines = reader.lines().collect::<Result<Vec<_>, _>>().unwrap();
```

### Testing

Simple test function, with assertions:

```rust
#[test]
fn my_test() {
  assert!(true);
  assert_eq!(1, 1);
  assert_ne!(1, 2);
}
```

### String/char-related

Conversions:

```rust
integer.to_string();                      // integer to string
let guess: u32 = string.parse().unwrap(); // string to numeric type
```

String APIs:

```rust
string.eq(&str)                           // test equality (compare)

string.clear();                           // blank a string
string.trim();                            // trim/strip
string.len();
string.as_bytes();                        // byte slice of the string contents
string.is_empty();                        // must be 0 chars long
string.push_str(&str);                    // concatenate (append) strings
string.push('c');
string.replace("a", "b");

string.split("sep")
string.split(char::is_numeric);
string.split(|c: char| c.is_numeric()).collect();
string.lines();                           // the newline char is not included in the output!
string.split_whitespace();

string += &string2;                       // concatenate via overloaded operator; can take &str or &String
format!("{}/{}/{}"), s1, s2, s3);         // preferred format for more complex concatenations
```

Char APIs:

```rust
c.is_alphabetic();
c.is_numeric();

c.to_uppercase();                         // returns an iterator (AAARGH!!!)

use std::char;

let c = char::from_digit(4, 10);          // (number, radix)
```

Formatting:

```rust
format!("The number is {}", 1);                          // the template *must* be a literal (!)
format!("The number is {0}, again {0}, not {1}!", 1, 2); // numbered placeholders!
```

### Random (`rand`)

```rust
use rand::Rng;

// `thread_rng()`: seeded by the O/S; local to the current thread.
// `gen_range()`: close,open ends.
//
let secret_number = rand::thread_rng().gen_range(0, 2);
```

### Regular expressions (`regex`)

```rust
let re = Regex::new(r"(\d{4})(\d{2})").unwrap();
let text = "201203, 201301, 201407";

re.is_match(text); // true

for cap in re.captures_iter(text) {
    println!("Whole: '{}', $1: '{}', $2: '{}'", &cap[0], &cap[1], &cap[2]);
}
```

### Date/times (standard)

```rust
// Compute time elapsed
//
let current_time = SystemTime::now();
current_time.elapsed();

// Get current time in seconds
//
let current_time_secs = SystemTime::now()
    .duration_since(UNIX_EPOCH)
    .unwrap()
    .as_secs();
```

### Date/times (`chrono`)

Don't use `time::Duration`, since it doesn't implement the operation traits.

```rust
let start:DateTime<Utc> = Utc::now();                   // Current time

DateTime::parse_from_str(string, "%d.%m.%Y %H:%M %P %z")?;   // Parse a datetime, with timezone
NaiveTime::parse_from_str(string, "%H:%M:%S")?;              // Parse a time
NaiveDateTime::parse_from_str(string, "%Y-%m-%d %H:%M:%S")?; // Parse a datetime, without timezone

start + Duration::days(3);                              // Arithmetic
start.checked_add(Duration::days(3));                   // Safe arithmetic
naive_time_1 + naive_time_2                             // Returns duration
nt1 + nt2 + Duration::seconds(nt3.second() as i64);     // Sample arithmetic with 3+ (odd) NaiveTime

// Conversions/tests

naive_time.second();                                    // requires `Timelike`
duration.num_seconds();

duration < Duration::zero();
```

### Commandline parsing (`clap`)

Example of varargs, and how to encapsulate the parsing logic:

```rust
fn parse_commandline_arguments<'a>(args: &'a Vec<String>) -> clap::ArgMatches {
    App::new("test")
        .setting(AppSettings::TrailingVarArg)
        .arg(
            Arg::with_name("INTERVALS")
                .required(true)
                .index(1)
                .multiple(true),
        )
        .get_matches_from(args)
}

fn extract_interval_arguments<'a>(matches: &'a clap::ArgMatches) -> Vec<&'a str> {
    matches
        .values_of("INTERVALS")
        .unwrap()
        .collect::<Vec<&str>>()
}

fn main() {
    let commandline_args = std::env::args().collect::<Vec<String>>();
    let matches = parse_commandline_arguments(&commandline_args);
    let intervals = extract_interval_arguments(&matches);
}
```

Examples of extended form:

```rust
let matches = App::new("My Super Program")
    .version("1.0")
    .author("Kevin K. <kbknapp@gmail.com>")
    .about("Does awesome things")
    .arg(
        Arg::with_name("config")
            .short("c")
            .long("config")
            .value_name("FILE")
            .help("Sets a custom config file")
            .takes_value(true),
    )
    .arg(
        Arg::with_name("INPUT")
            .help("Sets the input file to use")
            .required(true)
            .index(1),
    )
    .arg(
        Arg::with_name("v")
            .short("v")
            .multiple(true)
            .help("Sets the level of verbosity"),
    )
    .subcommand(
        SubCommand::with_name("test")
            .about("controls testing features")
            .version("1.3")
            .author("Someone E. <someone_else@other.com>")
            .arg(
                Arg::with_name("debug")
                    .short("d")
                    .help("print debug information verbosely"),
            ),
    )
    .get_matches();

// Gets a value for config if supplied by user, or defaults to "default.conf"
let config = matches.value_of("config").unwrap_or("default.conf");
println!("Value for config: {}", config);

// Calling .unwrap() is safe here because "INPUT" is required (if "INPUT" wasn't
// required we could have used an 'if let' to conditionally get the value)
println!("Using input file: {}", matches.value_of("INPUT").unwrap());

// Vary the output based on how many times the user used the "verbose" flag
// (i.e. 'myprog -v -v -v' or 'myprog -vvv' vs 'myprog -v'
match matches.occurrences_of("v") {
    0 => println!("No verbose info"),
    1 => println!("Some verbose info"),
    2 => println!("Tons of verbose info"),
    3 | _ => println!("Don't be crazy"),
}

// You can handle information about subcommands by requesting their matches by name
// (as below), requesting just the name used, or both at the same time
if let Some(matches) = matches.subcommand_matches("test") {
    if matches.is_present("debug") {
        println!("Printing debug info...");
    } else {
        println!("Printing normally...");
    }
}
```

Examples of compact form:

```rust
let matches = App::new("myapp")
    .version("1.0")
    .author("Kevin K. <kbknapp@gmail.com>")
    .about("Does awesome things")
    .args_from_usage(
        "-c, --config=[FILE] 'Sets a custom config file'
  <INPUT>              'Sets the input file to use'
  -v...                'Sets the level of verbosity'",
    )
    .subcommand(
        SubCommand::with_name("test")
            .about("controls testing features")
            .version("1.3")
            .author("Someone E. <someone_else@other.com>")
            .arg_from_usage("-d, --debug 'Print debug information'"),
    )
    .get_matches();
```