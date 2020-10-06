# Rust
- [Rust](#rust)
  - [Cargo](#cargo)
  - [Syntax/basics](#syntaxbasics)
    - [Basic structure/Printing/Input](#basic-structureprintinginput)
      - [Printing/formatting](#printingformatting)
    - [Variables/Data types](#variablesdata-types)
    - [Basic operators/operations/arithmetic/math](#basic-operatorsoperationsarithmeticmath)
    - [Closures/Functions](#closuresfunctions)
    - [Ranges and `std::iter::Iterator` methods](#ranges-and-stditeriterator-methods)
      - [Method chaining](#method-chaining)
      - [Iterator trait/Associated types](#iterator-traitassociated-types)
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
    - [Generics](#generics)
    - [Traits (and Generics #2)](#traits-and-generics-2)
    - [Traits #2 (OO-approach and supertraits)](#traits-2-oo-approach-and-supertraits)
    - [Traits #3 (disambiguation)](#traits-3-disambiguation)
    - [Operator overloading](#operator-overloading)
    - [[Static] Methods](#static-methods)
    - [Ownership](#ownership)
      - [Move](#move)
      - [Borrowing](#borrowing)
      - [Dangling pointers](#dangling-pointers)
      - [Lifetimes](#lifetimes)
      - [Slices](#slices)
    - [Smart pointers](#smart-pointers)
      - [Box<T>](#boxt)
      - [RC<T>](#rct)
      - [RefCell<T> and interior mutability](#refcellt-and-interior-mutability)
      - [Weak<T> and reference cycles](#weakt-and-reference-cycles)
    - [Multithreading](#multithreading)
      - [Channels: Multiple Producers Single Consumer](#channels-multiple-producers-single-consumer)
      - [Mutex<T>/Arc<T>](#mutextarct)
      - [Atomic primitive type wrappers](#atomic-primitive-type-wrappers)
      - [Barrier](#barrier)
      - [Condvar](#condvar)
      - [Busy waiting/spin loops](#busy-waitingspin-loops)
    - [Unsafe](#unsafe)
      - [Interoperability with other languages (C)](#interoperability-with-other-languages-c)
    - [Macros](#macros)
      - [Rules and details](#rules-and-details)
    - [Unions](#unions)
  - [Packaging](#packaging)
    - [Project structure](#project-structure)
    - [Modules](#modules)
  - [Standard library](#standard-library)
    - [Files/streams handling](#filesstreams-handling)
    - [Testing](#testing)
      - [Integration tests](#integration-tests)
    - [String/char-related](#stringchar-related)
    - [VecDeque: double-ended queue](#vecdeque-double-ended-queue)
    - [TCP client/server](#tcp-clientserver)
    - [Commandline arguments (basic)](#commandline-arguments-basic)
    - [Processes](#processes)
    - [Blackbox (nightly)](#blackbox-nightly)
  - [Traits](#traits)
    - [Default](#default)
    - [Copy, Clone, Drop and their relationships](#copy-clone-drop-and-their-relationships)
    - [Index[Mut]](#indexmut)
  - [Crates](#crates)
    - [Random (with and without `rand`)](#random-with-and-without-rand)
    - [Regular expressions (`regex`)](#regular-expressions-regex)
    - [Date/times (standard)](#datetimes-standard)
    - [Date/times (`chrono`)](#datetimes-chrono)
    - [Commandline parsing (`clap`)](#commandline-parsing-clap)
    - [Map literals (`maplit`)](#map-literals-maplit)
    - [Channels: Single Producer Multiple Consumers (`bus`)](#channels-single-producer-multiple-consumers-bus)
    - [Unit testing: demonstrate](#unit-testing-demonstrate)

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
# Add the member before creating the crate.

[workspace]
members = ["playground", "rust_programming_by_example"]

[dependencies]
redisish = {path = "../redisish"}   # Relative dependency
```

Versioning is pessimistic by default.

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.

## Syntax/basics

### Basic structure/Printing/Input

```rust
// "attributes": metadata with different purposes.
// with the `!`, they are at crate level (must be in the root crate); without, they are at method
// level (place them immediately above the method definition).
//
#![allow(dead_code)]
#![allow(unused_variables)]
#![allow(unused_imports)]
#![allow(unused_assignments)]
#![allow(unused_must_use)]
#![allow(unused_mut)]

use std::io;
use std::io::Write; // bring flush() into scope

// Unused parameters can be named `_`.
//
fn testing(n: u32) -> String {
  if n > 10 {
    panic!("Error message!");
  }
  // In order to return a value without using `return`, omit the semicolon.
  //
  String::from("abc")
}

// The return value is optional. This specific one is convenient [for testing].
//
fn main() -> std::result::Result<(), Box<dyn Error>> {
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

  // See fn return value.
  //
  Ok(());
}
```

#### Printing/formatting

```rust
println!("{:#?}", vec);                 // generic pretty printing
println!("{:?}", vec);                  // `Debug` format (requires the `Debug` trait; if generic, requires `use std::fmt::Debug`)

eprintln!("Error!");                    // print on stderr!

format!("The number is {}", 1);                          // the template *must* be a literal (!)
format!("The number is {0}, again {0}, not {1}!", 1, 2); // numbered placeholders!

writeln!("{}", buffer, 123);            // write formatted data into a buffer
```

Formatting (see https://doc.rust-lang.org/std/fmt):

```rust
"{:.2}"             // round float
"{:x}/{:X}"         // lower/upper hex
"{:b}"              // binary

"{:5}"              // padding (right align)
"{:05}"             // padding with char (zero)

"{:<5}"             // left align
"{:^5}"             // center align
"{:>5}"             // right align

"{:-^5X}"           // pad with char, align, and convert ot hex
```

In order to escape a curly brace, duplicate it.

In order to convert bytes to a hex string, use:

```rust
use core::fmt::Write;
let mut buffer = String::with_capacity(2 * array.len());
for byte in buffer {
  write!(s, "{:02X}", byte)?;
}
```

### Variables/Data types

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required

((1u128 << CONST_U64) - 1) as u64   // WATCH OUT the priorities! In this example, the brackets are all required!

static HELLO_WORLD: u32 = 1000;     // static variable

type Kilometers = i32;                     // type aliasing
type Result<T> = Result<T, std::io:Error>; // library example: `std::io::Result`

let bool_as_int = true as i32;      // true: 1, false: 0
let int_as_bool = 1 as bool;        // 1: true, 0: false, other: !!undefined!!
```

Numeric casts:

```rust
255_u8 as u16; // 255 ("zero-extend")
-1_i8 as u16;  // 65535 ("signed-extend")
```

SVs differ from constants:

- constants can be duplicated in memory;
- SVs can be mutable (in this case, they're unsafe).

Integer types:

- `[iu](8|16|32|64|128|size)`
  - the `size` ones depend on the architecture
  - the default, if not specified and can't be inferred, is `i32`

Max value functions (example): `u64::max_value()`, `u64::MAX`

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

// Use a tuple as function argument; borrowing is an option.
//
fn multiply(dimensions: &(u32, u32)) -> u32 {
  dimensions.0 * dimensions.1
}

multiply(&(2, 3));
```

For strings, see the [Strings chapter](#strings).

### Basic operators/operations/arithmetic/math

```rust
val += 1; val -= 1;             // increment/decrement value (no postfix)
val <<= n; val >>= n;           // overflows are ignored
std::mem::swap(&mut a, &mut b); // !! swap two variables !!

10_u64.pow(2);                  // exponentiation (power), int/int
10_f64.powi(2);                 // exponentiation, float/int
10_f64.sqrt();                  // square root
10_f64.sin();                   // sine (in rad)
10_f64.signum();                // positive: 1.0, negative: -1.0

std::f64::consts::PI;           // Pi

u32::max(1, 2);                 // maximum between two numbers
std::cmp::max(x, u);            // maximum between two numbers

z, carry = x.overflowing_add(y); // adds and wraps around in case of overflow; <carry> is bool.
z, carry = x.overflowing_sub(y); // subtracts and wraps around, as above
                                 // WATCH OUT! For other `overflow_` operations, `carry` may not the intuitive value, eg. for bit shift

(f * 100.0).round() / 100.0;    // round to specific number of decimals (ugly!!; also see #printing)
0_u32.to_be_bytes();            // convert big endian u32 to array of bytes
u32::from_le_bytes(&[u8])       // convert big endian array of bytes to u32
```

### Closures/Functions

Equivalent of Ruby blocks!

```rust
let multiple_of_10 = |x| { x % 10 == 0; }   // yay! note: the braces are optional
(0..100).any(multiple_of_10);               // double yay!
let is_0 = |x: i32| -> bool { x == 0; }     // with type annotations; they're not required

// Generic closure signature. The closure types passed don't need to be annotated.
//
struct Calculator<T: Fn(u32) -> u32>
{
  calculation: T,
}
```

Functions can be assigned to variables (function pointers); their type is `fn` (don't confuse with `Fn`!!). However, they can't reference the context, and functions defined inside a function can't be assigned to a variable.

```rust
fn sum_fn(x: i32) -> i32 {
  x
}

fn main() {
  let y = 10;

  fn sum_fn(x: i32) -> i32 { x + y };     // Invalid

  let my_fn = sum_fn;                     // Valid

  fn return_value_fn(x: i32) -> i32 { x } // Valid

  return_value_fn(1);
}

// Syntax to pass methods (both instance/static):

fn cycle_decode(&self) -> (fn(&Self, u8), u8) {
  let cycle_execute = Self::cycle_execute_foo;
  (cycle_execute, 0)
}

let (cycle_execute, x) = self.cycle_decode();
cycle_execute(self, x);
```

Because of the capturing, closures have overhead compared to functions.

Closures can use function pointers, including initializer functions!!:

```rust
[0, 1, 2, 3].iter().map(ToString::to_string);   // Associated method `to_string()` of the `ToString` trait
[0, 1, 2, 3].iter().map(Some);                  // Initializer function of the `Some` enum
```

The compiler performs the "Deref coercion", if required - essentially, a series of dereferencings (following the `Deref` trait):

```rust
fn hello_slice(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let pizza = Box::new(String::from("abc"));
    hello_slice(&pizza);
}
```

in the above case, both the box dereference, and the `String` deference that turns `&String` into `&str`.

Closure can have three traits, which are inferred:

- `FnOnce`: take ownership (which can't be taken more than "once")
- `FnMut`: borrow mutably
- `Fn`: borrow immutably

In order to return closures from a function, they must be boxed:

```rust
fn my_closure() -> Box<dyn Fn(i32) -> i32> {
  Box::new(|x| x + 1)
}
```

### Ranges and `std::iter::Iterator` methods

General form: `[start] .. [[=]end]`.

Ranges are:

- lazy;
- open ended on the `end`, unless `=` is specified.

Iterator getting methods:

```rust
collection.iter()            // immutable references
collection.iter_mut()        // mutable references
collection.into_iter()       // owned values
```

`std::iter::Iterator` methods, implemented by Range:

```rust
map(|x| x * 2)               // Ruby :map
map(|(x, y)| x + y)          // Tuples unpacking: useful for example, on the result of zip()
fold(a, |a, x| a + x)        // Ruby :inject
filter(|x| x % 2 == 0)       // Ruby :select
find(|x| x % 2 == 0)         // find first element matching the condition
rev()                        // reverse. WATCH OUT, UNINTUITIVE: since it's not inclusive, it goes from 99 to 0.
any(|x| x == 33)             // terminates on the first true
all(|x| x % 2 == 0)          // terminates on the first false
nth(n)                       // nth element (0-based)
take(n)                      // iterator for the first n elements
enumerate()                  // iterator (index, &value) (Ruby :each_with_index)
join("str")                  // join using str
zip(iter)                    // zip two arrays (iterators)!!!
sum()                        // WATCH OUT! Returns the same type, so conversion is needed, e.g. `.map(|&x| x as u32).sum();`

chunks(n)                    // iterate in chunks of n elements; includes last chunk, if smaller
chunks_exact(n)              // iterate in chunks of n elements; does not include the last chunk, if smaller
windows(n)                   // like chunks, but with overlapping slices

// transform an iterator into a collection ("consume")
collect()
collect::<Vec<i32>>()
collect::<Vec<_>>()

// Create an iterator for repeating a value
std::iter::repeat(x)
```

#### Method chaining

Chain iterators, AREL-style:

```rust
pub fn compose_iterator<T>(elements: &Vec<T>, reverse: bool) {
    let mut base_iter = elements.iter();
    let mut rev_iter;

    let mut composed_iter: &mut dyn Iterator<Item = &T> = &mut base_iter;

    if reverse {
        rev_iter = base_iter.rev();
        composed_iter = &mut rev_iter;
    }

    // ...
}
```

Additional iterators can be added following the same `rev_iter` pattern.

#### Iterator trait/Associated types

```rust
// Basic Iterator implementation.
//
impl Iterator for PhonyCounter {
  // Associated type. Similar to generics, however, doesn't require the type to be specified every
  // time the related methods (in this case, `next()`) are invoked.
  // In this case, a default is specified, which is optional.
  //
  type Item = u32;

  fn next(&mut self) -> Option<Self::Item> {
    Some(0)
  }
}
```

### Arrays/Vectors/Slices

Arrays (immutable, so they're allocated on the stack):

```rust
let my_list = [1, 2, 3];
let my_list = [true; 4];                // 4 elements initialized as true; won't work with variable size (use a Vec)
let my_list: [u32; 3] = [1, 2, 3];      // with data type annotation; ugly!
let mut my_list: [Option<u32>; 3] = [None; 3];  // with Option<T>; super-ugly!

my_list[512..].copy_from_slice(&source) // memcpy (copy) from/to slices/vectors; source/dest size must be the same!
my_list.fill(value)                     // memset; unstable as of Aug/2020

// invocation: process_list(&my_list)
//
fn process_list(list: &[i32]) {}
```

Vectors (mutable):

```rust
let mut vec = Vec::new();               // Basic (untyped) instantiation (if the type can be inferred)
let mut vec: Vec<i32> = Vec::new();     // Basic, if type can't be inferred
let mut vec = vec![1, 2, 3];            // Macro to initialize a vector from a literal list
let mut vec = vec![true; n];            // Same, with variable-specified length and initialization
Vec::with_capacity(cap);                // Preallocating version; WATCH OUT! The length is still 0 at start; use `vec![<val>, n]` if required

vec[0] = 2;

let val = &vec[0];
vec.get(2);                             // Safe (Option<T>) version
vec.first();
vec.last();

vec.push(1);                            // Push at the end
vec.pop();                              // Pop from the end
vec.swap(pos1, pos2);
vec.extend([1, 2, 3].iter().copied());  // Append (concatenate) a list
vec.extend(&[1, 2, 3]);                 // Append (borrowing version)
vec[range].copy_from_slice(&source);    // memcpy; see array example
(sl1, sl2) = vec.split_at(split_point); // immutably split an array, into two slices
vec2 = vec.split_off(split_point);      // mutably split an array: the second half is removed from `vec` and returned as new array

vec.len();
vec.iter();                             // iterator

// Pattern matching!
match v.get(2) {
  Some(entry) => println!("The third element is {}", entry),
  None => println!("There is no third element."),
}

// Vectors can be received as array reference type (mutable, if required).
//
fn process_list<T>(list: &[T]) {};
process_list(&vec);
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
let array = [8u32; 5];          // type is [u32; _]
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
// There's no function for getting with a default, but `[cloned()].unwrap_or()` works well.
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

In order to use enums as keys, annotate them with `#[derive(Eq, PartialEq, Hash)]`.

Map literals are not supported. See the maplit crate.

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

// Custom increment/decrement. See previous comment; goes from 98 to 0. It's not possible to write
// `.rev().step_by(-2)`, since `step_by()` takes a usize.
//
for x in (0..100).step_by(2).rev() {}

// Iterate an array/vector
//
for i in collection.iter() { }        // see #ranges-and-stditeriterator-methods for the other methods
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

// If let (can match the same way pattern matching does; see specific section).
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
//
// - `Debug`: allows printing (also required by asserts);
// - `PartialEq`: allows comparison with asserts;
// - `Eq`: good practice to implement if applies.
//
#[derive(Debug, PartialEq, Eq)]
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

// take(): extract a value and replace with None:
//
let mut x = Some(2);
let y = x.take();
x == None;
y == Some(2);
```


See next section for pattern matching.

### Pattern matching

```rust
// Generic match
//
let xxx = match val {
  1 | 2 => "1 or 2",
  3..5  => "between 3 and 5", // chars also allowed
  _     => {
    "other"
  }
};

// Match enum; all the entries must be exhausted (or the `_` placeholder must be used).
// The second case is a variant with an associated value!
//
// See `if let` for an alternative structure.
//
match message {
  Message::Move{x, y} => { self.position = Point { x: x, y: y } }
  Message::Echo(s) => { self.echo(s) }
  Message::ChangeColor(c1, c2, c3) => { self.color = (c1, c2, c3) }
  Message::Publish(Message(message)) => {} // nested!
  Message::None(_) => {}                   // ignore content
}

// Match tuples
//
match (2, 4, 8, 16, 32) {
  (first, .., last) => { // ignore values
    println!("Some numbers: {}, {}", first, last);
  }
};

// Match Option<T>
//
let y = 10;
match x {
  None              => None,
  Some(i)           => Some(i + 1),
  Some(x) if x == y => Some(10),    // "Match guard"
  Some(y)           => Some(y * 2), // Shadowing! Not what it looks!
};

// Match Result<T, Err>
//
match Url::parse(&line) {
  Ok(url) => println!("ok"),
  _ => println!("ko"),
};

// Match a struct (!!)
//
let point = Point { x: 1, y: 1 };
match point {
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y }    => println!("On neither axis: ({}, {})", x, y),
    Point { x, .. }   => {} // ignore rest of the struct

    // "Binding": assign a value while testing, via `@`.
    //
    Point { x: x_val @ 3..=7 } => println!("Found an x in range: {}", id_variable),
};

// Match an enum inside a struct (!!!):
//
if let Event::Window { win_event: WindowEvent::SizeChanged(new_width, new_height), .. } = event { /* .. */ }
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
  active: false,
};

// Update it. Requires the whole struct to be mutable!
user.active = true

// Destructuring a struct (!!)
let User {username: a, active: b} = user;
println!("a:{}, b:{}", a, b); // "a:sav, b:false"

// Hardcore destructuring
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
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

### Generics

"Monomorphization": turn generic types into concrete ones at compile time.

```rust
fn larger<T: PartialOrd>(a: T, b: T) -> bool {
  a > b
}

enum Option<T> {
  Some(T),
  None
}

struct Point<T, U> {
  x: T,
  y: U,
}

impl<T, U> Point<T, U> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

### Traits (and Generics #2)

```rust
pub trait Summary {
  fn summarize(&self) -> String;

  // Default method. Overriding doesn't require special syntax.
  //
  fn default_placeholder(&self) -> String {
    "Filomegna donde estas"
  }
}

pub struct Article {
  pub text: String,
}

// Traits can be imlemented only if the trait or the type are local to the crate!
// This way, one doesn't find "surprises" from other crates.
//
impl Summary for Article {
  fn summarize(&self) -> String {
    "Summary of an article!".to_string()
  }
}

// Specify trait as type. Can specify more (e.g. <T: Summary + Display>).
// The alternative syntax is `print_summary(text: impl T) -> impl T`.
//
fn print_summary<T: Summary>(text: T) -> T {
  println!("{}", text.summarize());
  text
}

// Returning different types is not allowed this way!
//
fn get_summary_type<T: Summary>(switch: bool) -> T {
  if switch {
    Article { text: "article".to_string() }
  } else {
    Tweet { text: "tweet".to_string() }
  }
}

// `where`: syntactic sugar.
//
fn fancy_types_function<T, U>(t: T, u: U)
where
    T: Display + Clone,
    U: Clone + Debug,
{}

// Unsafe trait and implementation
//
unsafe trait Foo {}
unsafe impl Foo for i32 {}

fn main() {
  let article = Article {
    text: "news".to_string(),
  };

  print_summary(article);
}
```

Things to keep in mind when writing generic functions:

```rust
// With this specific structure, T needs to implement Copy because:
//
// - Returning the first element is a copy operation
// - iter() also returns copies
//
// An alternative not using references is Clone, which is expensive.
//
fn first_element<T: Copy>(list: &[T]) -> T {
  for &item in list.iter() {}
  list[0]
}
```

Conditional ("blanket") implementations (uses `Point` from [Generics](#generics)):

```rust
// Implement only for `Point`s whose T implements Display and Ord.
//
impl<T: Display + Ord, U> Point<T, U> {
  fn greatest_x(&self, other: &Point<T, U>) {
    let max_x = std::cmp::max(&self.x, &other.x);
    println!("Largest x: {}", self.x);
  }
}
```

### Traits #2 (OO-approach and supertraits)

```rust
pub trait Draw { fn draw(&self); }

pub struct Screen<T: Draw> {
  pub components: Vec<T>,
}
impl<T: Draw> Screen<T> { pub fn run(&self) {} }

pub struct Pizza {}
impl Draw for Pizza { fn draw(&self) {} }

pub struct Tapparella {}
impl Draw for Tapparella { fn draw(&self) {} }

Screen {
  components: vec![Pizza {}, Pizza {}],
};
```

Dynamic dispatch version. components needs to be declared as `Vec<Box<dyn Draw>>`, where `dyn` indicates dynamic dispatch.

```rust
pub trait Draw { fn draw(&self); }

pub struct Screen {
  pub components: Vec<Box<dyn Draw>>,
}
impl Screen { pub fn run(&self) {} }

pub struct Pizza {}
impl Draw for Pizza { fn draw(&self) {} }

pub struct Tapparella {}
impl Draw for Tapparella { fn draw(&self) {} }

Screen {
  components: vec![Pizza {}, Tapparella {}],
};
```

`dyn <type>`s are called "trait objects". They require a pointer, like `Box<T>` or a reference (`&`); the methods in the thread must be "object safe":

- return type isn't self;
- no generics.

In order to specify a dynamic type when boxing, must explicitly cast:

```rust
Box::new(NullLogger {});                    // type = Box<NullLogger>
Box::new(NullLogger {}) as Box<dyn Logger>; // type = Box<dyn Logger>
```

Supertraits are traits depending on other traits:

```rust
// Example using default implementation.
//
trait BetterDisplay: fmt::Display {
    fn better_to_string(&self) -> String {
        format!("Better!: {}", self.to_string())
    }
}
```

### Traits #3 (disambiguation)

Specify which function to invoke, when there is overlapping with/between traits:

```rust
trait Flyer {
  fn fly(&self);
  fn mean() -> String;          // associated method
}

struct Human;

impl Flyer for Human {
  fn fly(&self) {
    println!("Flying!");
  }
  fn mean() -> String {
    "Plane".to_string()
  }
}

impl Human {
  fn fly(&self) {
    println!("No way");
  }
  fn mean() -> String {
    "Arms, hopefully".to_string()
  }
}

let person = Human;

person.fly();        // "No way"
Flyer::fly(&person); // "Flying"

println!("{}", Human::mean());            // "Arms, hopefully"
println!("{}", <Human as Flyer>::mean()); // "Plan"
println!("{}", Flyer::mean());            // error!
```

### Operator overloading

```rust
struct Point(i32);
struct BigPoint(i32);

impl Add for Point {
  type Output = Point;

  // `rhs` stays for `Right Hand Side`
  //
  fn add(self, rhs: Self) -> Self::Output {
    Point(self.0 + rhs.0)
  }
}

impl Add<BigPoint> for Point {
  type Output = Point;

  fn add(self, rhs: BigPoint) -> Self::Output {
    Point(self.0 + 1000 * rhs.0)
  }
}

Point(10) + Point(20)
Point(10) + BigPoint(1)
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

### Ownership

#### Move

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

#### Borrowing

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

#### Dangling pointers

In Rust, it's not possible to have dangling pointers:

```rust
// Invalid!
//
fn dangling() -> &String {
  &String::from("text");
}

// Must return the string directly.
//
// Valid!
//
fn not_dangling() -> String {
  String::from("text");
}
```

#### Lifetimes

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

Structs can have lifetimes, but they must be applied to all the members.

```rust
struct ImportantExcerpt<'a> {
  part: &'a str,
}

// Syntax for implemented methods:
//
impl<'a> ImportantExcerpt<'a> {
  fn level(&self) -> i32 { 3 }
}
```

In some cases, Rust can infer lifetimes, so they don't need to be specified.

Syntax notes:

```rust
// lifetimes + mutable + generics
//
fn method<'a, T>(var: &'a mut T) {
  // The `'static` lifetime lasts for the entire program execution (typically, hardcoded strings).
  //
  let str: &'static str = "hardcoded";
}
```

#### Slices

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

### Smart pointers

Smart pointers implement the traits:

- `Deref`: makes instances behave like pointers (implementing the deference operator (`*`));
- `Drop`: invoked when instances go out of scope.

In order to manually drop an instance, use `std::mem::drop`.

#### Box<T>

Allow storing data on the heap rather than on the stack; useful for:

- a type whose size can’t be known at compile, but it's required, e.g. recursive types;
- data size constitutes a performance problem (if copied);
- owning a value when a trait is required, without forcing it to be of a specific type.

Generic usage:

```rust
{
  let boxed = Box::new(5);

  // Dereference, in order to use the boxed value.
  //
  assert_eq!(5, *boxed);

  // here the pointer and data are deallocated
}
```

Use in recursive structures:

```rust
// Without Box, the compiler complains that indirection is needed.
//
enum Node {
  Parent(Box<Node>),
  Nil,
}

use self::Node::{Nil, Parent};

let parent_a = Parent(Box::new(Parent(Box::new(Nil))));
```

#### RC<T>

"Reference Counting"; use when there are multiple owners, e.g. graphs, that is, when the compiler can't know who will use the reference last.

Can hold only immutable values; can be used only on single thread contexts.

```rust
enum Node {
  Parent(Rc<Node>),
  Nil,
}

use self::Node::{Nil, Parent};
use std::rc::Rc;

// Invalid; immutable-only!
//
let value = Rc::new(1);
*value = 32;

// `Rc::clone()` is shallow; use `clone()` for deep copies.
//
let a = Rc::new(Parent(Rc::new(Parent(Rc::new(Nil)))));
{
  let _b = Parent(Rc::clone(&a));
  let _c = Parent(Rc::clone(&a));
  println!("Count: {}", Rc::strong_count(&a)); // 3
}
println!("Count: {}", Rc::strong_count(&a)); // 1
```

#### RefCell<T> and interior mutability

`RefCell<T>` allows mutating the contained value, even if the variable itself is immutable, therefore bypassing the compiler; generally speaking, it allows multiple owners while retaining mutability. The rules are enforced at runtime though, so this has an overhead.

```rust
enum Node {
    Parent(Rc<RefCell<i32>>, Rc<Node>),
    Nil,
}

use self::Node::{Nil, Parent};
use std::cell::RefCell;
use std::rc::Rc;

let value = Rc::new(RefCell::new(1));

let a = Rc::new(Parent(Rc::clone(&value), Rc::new(Nil)));
let _b = Parent(Rc::new(RefCell::new(2)), Rc::clone(&a));
let _c = Parent(Rc::new(RefCell::new(3)), Rc::clone(&a));

*value.borrow_mut() += 10;
```

#### Weak<T> and reference cycles

Full tree data structure, with nodes pointing both to children and parents. The problem is that if we don't use weak references, there will be circular references (therefore leaks) because of parents pointing to children, and viceversa.

It's important to always thing who is the owner. A parent ultimately owns the children - if the former is dropped, the children should be dropped too; therefore, the parent reference should be weak.

In order to access a `Weak<T>` value, call `upgrade() -> Option<T>`.

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

let leaf = Rc::new(Node {
    value: 3,
    parent: RefCell::new(Weak::new()),
    children: RefCell::new(vec![]),
});

println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

let branch = Rc::new(Node {
    value: 5,
    parent: RefCell::new(Weak::new()),
    children: RefCell::new(vec![Rc::clone(&leaf)]),
});

*leaf.parent.borrow_mut() = Rc::downgrade(&branch);

println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
```

There are methods to perform counting:

```rust
// Example, referencing branch.
//
Rc::strong_count(&branch);
Rc::weak_count(&branch);
```

### Multithreading

Base usage:

```rust
use std::thread;
use std::time::Duration;

let message = "Hello";

// `move` is required to access variable in the context, with move semantics.
//
let handle = thread::spawn(move || {
  println!("{}", message);
  thread::sleep(Duration::from_millis(1));
});

handle.join().unwrap();
```

#### Channels: Multiple Producers Single Consumer

For SPMC, see crate [bus](#channels-single-producer-multiple-consumers-bus), although it can be emulated with multiple channels.

```rust
use std::sync::mpsc;

// Sender channels can be cloned ("Multiple Producer Single Consumer").
//
// For synchronous channels (bound by a buffer), use mpsc::sync_channel(bufsize). A buffer size of
// 0 blocks until a receiver reads.
//
let (tx1, rx) = mpsc::channel();
let tx2 = mpsc::Sender::clone(&tx1);

thread::spawn(move || {
  let val = "hi";
  // send() doesn't block
  tx1.send(val).unwrap();
  // `val` is now moved; can't be used anymore
});

thread::spawn(move || {
  tx2.send("hello").unwrap();
});

// Non-blocking version: `rx.try_recv()`.
//
println!("{}", rx.recv().unwrap());

for received in rx {
  println!("{}", received);
}
```

#### Mutex<T>/Arc<T>

```rust
// We need to use an "Atomic Reference Counter" because the counter is shared between threads;
// the compiler interprets the counter usage as multiple move.
//
let counter = Arc::new(Mutex::new(0));

for _ in 0..10 {
  let counter = Arc::clone(&counter);

  thread::spawn(move || {
    let mut num = counter.lock().unwrap();
    *num += 1;
  });
}

// threads must be joined
```

#### Atomic primitive type wrappers

Primitive types have special atomic structs that are very efficient, e.g. (simplified):

```rust
// AtomicU128 is only in nightly.
//
let current_cycle = Arc::new(AtomicU32::new(0));

{
    let current_cycle = current_cycle.clone();

    thread::spawn(move || {
      let mut cycle_to_execute = 0;

      while cycle_to_execute < cycles {
        while cycle_to_execute <= current_cycle.load(Ordering::Relaxed) {
          // some work
          cycle_to_execute += 1;
        }
      }
    })
}

for cycle_i in 0..cycles {
  current_cycle.store(cycle_i, Ordering::Relaxed);
}
```

Some APIs (return the previous value):

- `fetch_add(value, ordering)`:                add
- `compare_and_swap(current, value, ordering`: if the existing value equals `current`, set to `value`

**WATCH OUT**: Check out the [CPP reference](https://en.cppreference.com/w/cpp/atomic/memory_order) to understand memory orderings.

#### Barrier

Enable multiple threads to start synchronized; `wait()` will block on each thread until the number has been reached.  
A random thread is the elected leader (`is_leader() == true`).  
After the barrier is released, it can be reused.

```rust
  let threads_number = 10;
  let barrier = Arc::new(std::sync::Barrier::new(threads_number));

  for _ in 0..threads_number {
    let barrier = barrier.clone();

    thread::spawn(move || {
      let barrier_wait_result = barrier.wait();
      barrier_wait_result.is_leader();
      my_operation();
    });
  }
```

#### Condvar

A [condvar](https://doc.rust-lang.org/beta/std/sync/struct.Condvar.html) allows threads to wait, and then be notified when a condition is met.

Watch out! This can't be used for as a pseudo-channel, because the mutex value could be changed between the notification, and the time the thread resumes. Example:

```rust
  let pair = Arc::new((Mutex::new(0), Condvar::new()));

  let handle = {
    let pair = pair.clone();

    thread::spawn(move || {
      let (lock, cvar) = &*pair;

      loop {
        let cycle_number_mutex = lock.lock().unwrap();

        // some work

        if *cycle_number_mutex == cycles - 1 {
          break;
        } else {
          cvar.wait(cycle_number_mutex).unwrap();
        }
      }
    })
  };

  let (lock, cvar) = &*pair;

  for cycle_number in 0..cycles {
    let mut mutex_cycle_number = lock.lock().unwrap();
    *mutex_cycle_number = cycle_number;
    cvar.notify_one();
  }

  handle.join();
```

#### Busy waiting/spin loops

Intrinsic to use in busy waiting:

```rust
std::sync::atomic::spin_loop_hint();
```

### Unsafe

Raw pointers:

```rust
fn print_raw_pointers(ref_mut: *mut i32, ref_const: *const i32) {
  unsafe {
    println!("r1:{}, r2:{}", *ref_const, *ref_mut);
  }
}

let mut num = 5;

// Creating pointers is not unsafe - only dereferencing them.
//
let ref_mut = &mut num as *mut i32;
let ref_const = &num as *const i32;

print_raw_pointers(ref_mut, ref_const);

// Invalid: (currently) this can't be done, in order to work around the BC - the variable needs to be
// declared (separately).
//
mymethod(&mut num, &num as *const i32);
```

Functions:

```rust
// Unsafe function; unsafe calls themselves don't require the enclosing function to be marked as such.
//
unsafe fn dangerous() {}

// Unsafe funcion call
//
unsafe {
  dangerous();
}
```

Example: slicing an array. It can't be done safely, because we can't mutably borrow twice.

```rust
let slice = &mut [0, 1, 2, 3][..];
let mid = 2;

let len = slice.len();
let ptr = slice.as_mut_ptr();

let (slice1, slice2) = unsafe {
  (
    std::slice::from_raw_parts_mut(ptr, mid),
    std::slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
  )
};
```

Referencing raw memory (undefined behavior):

```rust
let address = 0x012345usize;
let r = address as *mut i32;

let slice: &[i32] = unsafe {
  std::slice::from_raw_parts_mut(r, 10000)
};
```

#### Interoperability with other languages (C)

Invoking C code:

```rust
// `"C"` defines the ABI, in this case, the C one.
//
extern "C" {
  fn abs(input: i32) -> i32;
}

unsafe {
  println!("Absolute value of -3 according to C: {}", abs(-3));
}
```

Allow code to be invoked from C:

```rust
// Requires disabling mangling.
//
#[no_mangle]
pub extern "C" fn call_from_c() { }
```

### Macros

Simple, fixed expressions:

```rust
// Capture the `expr`ession as `$name`.
//
macro_rules! simple_print {
    ($name:expr) => {
        println!("Hey {}", $name)
    };
}

// `envar!(setValue: "/home/me" forKey: "HOME")`
//
macro_rules! envar {
    (setValue: $value:tt forKey: $key:tt) => {
        println!("setValue:{} forKey:{}", $value, $key);
    };
}

// Multiple expressions for one macro:
//
// - `add!(one to 5)`
// - `add!(two to 5)`
//
macro_rules! add {
    (one to $input:expr) => {
        $input + 1
    };
    (two to $input:expr) => {
        $input + 2
    };
}
```

Macros can also be invoked using both round and square brackets.

Multiple occurrences. Metachars are:

- `*`: 0+
- `+`: 1+
- `?`: 0 or 1 (doesn't take a separator)

```rust
// The character preceding `*` is the separator.
//
macro_rules! repeated_print {
    ($( $name:expr ),*) => {
      $( println!("Hey {}", $name); )*
    };
}

// This way repeats only the expressions.
//
macro_rules! build_and_print_array {
  ($( $item:expr ),*) => {
    let myvec = vec![$( $item ),*];
    println!("{:?}", myvec);
  };
}

// Accepts expressions with the Ruby hash (rocket) syntax.
// Since we return a value, we need to define a scope (extra curly braces).
//
macro_rules! ruby_hash {
    ($( $key:expr => $value:expr ),*) => {
        {
            let mut hm = HashMap::new();
            $( hm.insert($key, $value); )*
            hm
        }
    };
}

// Simulate an optional parameter.
// Different use of the comma here; it's consumed as part of the expression, but the expanded one is
// the one defined by the repetition.
//
macro_rules! optional_param {
    ($mandatory:expr $(, $optional:expr)*) => {
        println!("M:{} O:{}", $mandatory, $($optional),*)
    };
}

// Simulate optional, named, parameters.
// For simplicity, the comma after each expression is mandatory.
//
//     test_cpu_execute!(
//         zf:80=>81,
//         hf:82=>83,
//     );
//
macro_rules! test_cpu_execute {
    (
        $( zf: $zf_pre_value:literal => $zf_post_value:literal, )?
        $( nf: $nf_pre_value:literal => $nf_post_value:literal, )?
        $( hf: $hf_pre_value:literal => $hf_post_value:literal, )?
        $( cf: $cf_pre_value:literal => $cf_post_value:literal, )?
    ) => {
        $( println!("zf: {} -> {}", $zf_pre_value, $zf_post_value); )?
        $( println!("nf: {} -> {}", $nf_pre_value, $nf_post_value); )?
        $( println!("hf: {} -> {}", $hf_pre_value, $hf_post_value); )?
        $( println!("cf: {} -> {}", $cf_pre_value, $cf_post_value); )?
    };
}
```

Other usages:

```rust
// Define a function (!!).
// If we don't define `$name` as `ident`, but for example as `expr`, the compiler will complain that
// an identifier is expected.
//
macro_rules! generate_func {
    ($name:ident) => {
        fn $name(param: i32) {
            println!("Value: {}", param);
        }
    };
}

generate_func!(foo);
foo(123); // `Value: 123`

// Define variables (!)
//
macro_rules! vars {
    ($data:expr, $stride:expr, $var1:ident, $var2:ident, $var3:ident) => {
        let $var1 = $data[0];
        let $var2 = $data[1 * $stride];
    };
}

// Pass self
//
macro_rules! call_on_self {
    ($self:ident, $F:ident) => {
        $self.$F()
    };
}
```

#### Rules and details

There are some rules:

- Statements and expressions can only be followed by `=>`, a comma, or a semicolon
- (other)

Macro variable types:

`expr`: Expressions that you can write after an `=` sign, such as `76+4` or `if a==1 {"something"} else {"other thing"}`.
`ident`: An identifier or binding name, such as `foo` or `bar`.
`path`: A qualified path. This will be a path that you could write in a use sentence, such as `foo::bar::MyStruct` or `foo::bar::my_func`.
`ty`: A type, such as `u64` or `MyStruct`. It can also be a path to the type.
`pat`: A pattern that you can write at the left side of an `=` sign or in a match expression, such as `Some(t)` or `(a, b, _)`.
`stmt`: A full statement, such as a `let` binding like `let a = 43;`.
`block`: A block element that can have multiple statements and a possible expression between braces, such as `{vec.push(33); vec.len()}`.
`item`: What Rust calls items. For example, function or type declarations, complete modules, or trait definitions.
`meta`: A meta element, which you can write inside of an attribute (`#[]`). For example, `cfg(feature = "foo")`.
`tt`: Any token tree that will eventually get parsed by a macro pattern, which means almost anything. This is useful for creating recursive macros, for example.
`literal`

Interesting articles:

- Tutorial: https://hub.packtpub.com/creating-macros-in-rust-tutorial
- Case study: https://notes.iveselov.info/programming/time_it-a-case-study-in-rust-macros

### Unions

```rust
// Watch out, x86 is little endian!
//
#[derive(Copy, Clone)]
struct Register8Pair {
    l: u8,
    h: u8,
}

// `#[repr(C)]` is necessary, to ensure that all the fields start at the same location.
//
#[repr(C)]
union Register16 {
    r16: u16,
    r8: Register8Pair,
}

let mut ax = Register16 { r16: 0xCAFE };

// Access is unsafe, because it's effectively undefined behavior.
unsafe { println!("AX:{:4X} AH:{:2X} AL:{:2X}", ax.r16, ax.r8.h, ax.r8.l) };

// Unsafe is not required for writes to `Copy` union fields.
ax.r8.l = 0xFF;

// Borrowing rules still apply: the first invocation is valid, but not the second!
//
fn test(_r1: &mut u8, _r2: &mut u8) {}
fn test2(_r1: &mut u16, _r2: &mut u8) {}
unsafe { test(&mut ax.r8.h, &mut ax.r8.l) };
unsafe { test2(&mut ax.r16, &mut ax.r8.l) };
```

Pattern matching works as usual, but exactly one field must be specified.

See [reference](https://doc.rust-lang.org/reference/items/unions.html#pattern-matching-on-unions) for the so-called "tagged" unions.

A proposal for ["unnamed" fields](https://rust-lang.github.io/rfcs/2102-unnamed-fields.html) has been approved, but not yet implemented.

## Packaging

### Project structure

A package (=set of 1+ crates) can contain at most one library crate.

Crate "roots":

- `src/main.rs` -> binary crate (same name as package)
- `src/lib.rs` -> library crate (same name as package)

Multiple crates can be put in `src/bin`:

- `src/bin/mycrate.rs` -> binary crate (named `mycrate`)

Alternative configuration for multiple crates, via `cargo.toml`:

```toml
# Array of tables -> there can be multiple.
#
[[bin]]
name = "daemon"
path = "src/daemon/bin/main.rs"
```

### Modules

All items (including modules) and their children are private by default, but:

- children can access their parents' items;
- public enums' items are public.

```rust
mod front_of_house {
  // Public module; doesn't make the *contents* public.
  //
  pub mod hosting {
    // Can see stuff in `front_of_house`.
    //
    pub fn add_to_waitlist() {}
  }
}

pub fn eat_at_restaurant() {
  // Relative path.
  // With this tree, the function invoked must be public.
  //
  front_of_house::hosting::add_to_waitlist();

  hosting::add_to_waitlist();
}
```

Modifiers (prefixes):

- `crate`: absolute path
- `super`: parent module
- `self`

Multiple files structure:

```rust
// Load the content, as a module.
//
// - if in the root crate, it will look into `<module_name>.rs`;
// - otherwise, `<filename_without_prefix>/<module_name>.rs`
//
mod <module_name>;
```

Importing:

```rust
// The item imported must be public. Using doesn't make the item public!
// In order to import relatively, must (currently) use `self`.
//
// It's unidiomatic to import functions, while it's idiomatic to import enums, structs, etc.
//
//
use crate::front_of_house::hosting;

// Solutions to clashing; both iditiomatic.
//
use std::io::Result as Ioresult;
use std::io; // and reference `io::Result`

// "Re-exporting"; allows referencing `hosting::add_to_waitlist`. Useful when the whole path is not
// meaningful for the clients.
//
pub use crate::front_of_the_house::hosting;

// Other use syntaxes
//
use std::io::{self, Write}
use std::collections::*; // useful for testing; unidiomatic for the rest
```

## Standard library

### Files/streams handling

```rust
std::fs::read_to_string(filename) -> Result<String, Error>; // content must be valid UTF-8; filename can be relative.
std::fs::read(game_rom_filename) -> Result<Vec<u8>, Error>; // read binary content

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

Unit testing:

```rust
fn my_method() -> u32 {
  1
}

#[cfg(test)] // Comple and run only in test mode
mod tests {
  // If really required, since this is a child module, this allows private functions to be tested.
  //
  use super::*;

  #[test]
  fn test_my_method() {
    assert_eq!(my_method(), 1);
    assert_ne!(my_method(), 0);
    assert!(my_method() == 1, "Value not matching!");
  }

  #[test]
  #[should_panic(expected = "yikes!")]
  fn test_my_method_panic() {
    panic!("yikes!")
  }

  #[test]
  fn test_with_result() -> Result<(), &'static str> {
    if true {
      Ok(())
    } else {
      Err("Error!")
    }
  }

  #[test]
  #[ignore] // Not included unless specified
  fn slow_test() {
    thread::sleep(Duration::from_secs(1));
  }
}
```

Cargo options:

```sh
cargo test $test_name_substring      # no patterns/regexes supported
cargo test -- --test=test-threads=1  # run serially (default is parallel)
cargo test -- --nocapture            # don't steal stdout output
cargo test -- --ignored              # include ignored tests
```

#### Integration tests

Integration tests are placed in the standard location `$project_root/tests`; run using `cargo test`.

```rust
// `#[cfg(test)]` is not required (assumed because of the directory), but the import is, because
// integration tests are not in scope
//
use mylibrary;

#[test] // blahblah
```

If one wants to write a shared module, put it in a submodule (eg. file `tests/common/mod.rs`), otherwise, it's interpreted as integration test suite.

When testing binary crates, don't forget that binary crates can't expose functions to be used by other crates (one of the reasons why `main.rs` is canonically thin an imports `lib.rs`).

### String/char-related

Conversions:

```rust
integer.to_string();                      // integer to string
let guess: u32 = string.parse().unwrap(); // string to numeric type
```

String APIs:

```rust
s.eq(&str)                              // test equality (compare)
s.len();
s.is_empty();                           // must be 0 chars long
s.contains("pattern");
s.start_with("pref");

s += &s2;                               // concatenate via overloaded operator; can take &str or &String
s.push_str(&str);                       // concatenate (append) strings
s.push('c');
s.to_lowercase(); s.to_uppercase();
s.replace("a", "b");                    // gsub
s.clear();                              // blank a string

s.trim(); s.trim_end(); s.trim_start(); // trim/strip
s.trim_end_matches("suffix");           // chomp suffix (but repeated)! also accepts a closure

s.as_bytes();                           // byte slice (&[u8]) of the string contents

// splits; there is a `r`split* version for each.
//
s.split("sep")
s.split(char::is_numeric);
s.split(|c: char| c.is_numeric()).collect();
s.splitn(max_splits, "sep").collect::Vec<T>(); // splits from left by separator to Vec, long `max_splits` maximum

s.lines();                              // the newline char is not included in the output!
s.split_whitespace();

format!("{}/{}/{}"), s1, s2, s3);       // preferred format for more complex concatenations
```

Char APIs:

```rust
c.is_alphabetic();
c.is_numeric();

c.to_lowercase(); c.to_uppercase();       // returns an iterator (AAARGH!!!)

use std::char;

let c = char::from_digit(4, 10);          // (number, radix)
```

### VecDeque: double-ended queue

```rust
let mut mailbox = VecDeque::new();
mailbox.push_back(item);
let front_item: Option<T> = mailbox.pop_front();
```

### TCP client/server

Client:

```rust
let mut read_buffer = String::new();
let mut connection = TcpStream::connect("127.0.0.1:8080")?;

connection.write_all("Hello".as_bytes())?; // requires `std::io::Read`

connection.shutdown(Shutdown::Write)?; // Read/Write/Both

let bytes_read = connection.read_to_string(&mut read_buffer)?; // requires `std::io::Write`
```

Server:

```rust
let listener = std::net::TcpListener::bind("127.0.0.1:8080")?;
for stream in listener.incoming() { handle_client(stream?) } // Watch out (Result!)
```

### Commandline arguments (basic)

```rust
std::env::args();     // only valid Unicode
std::env::args_os();  // returns `OsString`s, which are not restricted to Unicode
```

### Processes

```rust
std::process::exit(exit_status);    // terminate program (exit)
```

### Blackbox (nightly)

Be pessimistic about the side effects of this function. Can't make any absolute guarantee.

```rust
#![feature(test)]

use test::bench::black_box;

pub fn black_box<T>(dummy: T) -> T
```

## Traits

### Default

```rust
// Generates a `::default()` method that fills the fields with the default values (0 for numeric, false for bool).
//
#[derive(Default)]
struct SomeOptions {
  foo: i32,
  bar: bool,
}

// Override the defaults.
//
SomeOptions { foo: 42, ..Default::default() };
```

### Copy, Clone, Drop and their relationships

See:

- https://stackoverflow.com/questions/51704063/why-does-rust-not-allow-the-copy-and-drop-traits-on-one-type
  - copy of `Copy` data is done via trivial `memcpy`; if drop was performed on a `Copy`+`Drop` copy, the original instance could include reference to invalid (not cleaned up) data
- https://www.reddit.com/r/rust/comments/8laxam/why_does_copy_require_clone
  - `Clone` is a supertrait of `Copy`

### Index[Mut]

Allow convenient map-like access to a struct.

```rust
struct Registers {
    SP: u16,
    PC: u16,
}

impl Index<Register> for Registers {
    type Output = u16;

    fn index(&self, register: Register) -> &Self::Output {
        match register {
            Register::SP => &self.SP,
            Register::PC => &self.PC,
        }
    }
}

impl IndexMut<Register> for Registers {
    fn index_mut(&mut self, register: Register) -> &mut Self::Output {
        match register {
            Register::SP => &mut self.SP,
            Register::PC => &mut self.PC,
        }
    }
}

// Access
//
let addr = self.registers[src_register] as usize;
self.registers[dst_register] = 0x21;
```

## Crates

### Random (with and without `rand`)

Poor man's random:

```rust
extern "C" {
  fn srand() -> u32;
  fn rand() -> u32;
}

unsafe {
  srand();
  println!("{}", rand());
}
```

With crate:

```rust
// Simplest way
//
let rand_byte: u8 = rand::random();

// Use thread_rng
// `thread_rng()`: seeded by the O/S; local to the current thread.
//
use rand::prelude::*;
let mut rng = rand::thread_rng();
let myrnd: f64 = rng.gen();
let myrnd: i32 = rng.gen_range(0, 2); // close,open ends.

let mut data = [0u8; 32];
rand::thread_rng().fill_bytes(&mut data);
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
// Compute time elapsed.
//
let current_time: SystemTime = SystemTime::now();
current_time.elapsed();

// Get current time in seconds
//
let current_time_secs = SystemTime::now()
    .duration_since(UNIX_EPOCH)
    .unwrap()
    .as_secs();
```
`SystemTime` is not monotonic; `Instant` is, which is important for gameloops, benchmarks etc. See [Instance manpage](https://doc.rust-lang.org/std/time/struct.Instant.html) for the details.

Simplified game loop:

```rust
let cycle_start_time = Instant::now();
some_work();
let next_cycle_time = cycle_start_time + Duration::new(0, 1_000_000_000 / 500);

// Remember the sleep time is the *minimum* quantity. For very small sleeps (e.g. 0.1ms), a sleep
// of 2.5 times as specified has been observed; this is expected in O/S implementations.
//
thread::sleep(next_cycle_time - Instant::now());
```

WATCH OUT! While `Instant`s can be compared, it's not possible to perform an operation whose result is negative.

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

Notes:

- In order to check boolean flags, use `ArgMatches#is_present(<str>)`;
- `Arg#takes_value(<bool>)` defaults to false;
- `Arg#index(<int>)` indicates the position of the argument (for positional arguments).

Example of varargs, partially encapsulated:

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

Example, fully encapsulated, but owned:

```rust
fn decode_commandline_args() -> Vec<String> {
  let commandline_args = std::env::args().collect::<Vec<String>>();

  let matches = App::new("test")
    .setting(AppSettings::TrailingVarArg)
    .arg(
      Arg::with_name("INTERVALS")
        .required(true)
        .index(1)
        .multiple(true),
    )
    .get_matches_from(commandline_args);

  matches
    .values_of("INTERVALS")
    .unwrap()
    .map(|arg| arg.to_string())
    .collect::<Vec<String>>()
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

### Map literals (`maplit`)

```rust
#[macro_use] extern crate maplit;

let map = hashmap!{
    "a" => 1,
    "b" => 2,
};
```

### Channels: Single Producer Multiple Consumers (`bus`)

```rust
// Buffer size. zero causes undefined behavior.
// Messages are put into the buffer, and the buffer is emptied when all the readers have read.
// There is a sweet spot in the buffer size for the performance.
// Performance need to be verified (in release mode) against the MPSC analog.
//
let mut tx = bus::Bus::new(1);

let mut rx1 = tx.add_rx();
let mut rx2 = tx.add_rx();

tx.broadcast("Hello");

thread::spawn(move || {
  println!("{}", rx1.recv().unwrap());
});

thread::spawn(move || {
  println!("{}", rx2.recv().unwrap());
});
```

### Unit testing: demonstrate

```rust
use demonstrate::demonstrate;

demonstrate! {
  use super::*;

  describe "test module 2" {
    before { let context = 5; }
    subject { context + 5 }

    it "test subject" {
      assert_eq!(subject, 10);
    }

    context "context is 5" {
      it "should equal 5" {
        assert_eq!(context, 5);
      }
    }
  }
}
```

The `use super::*` is not needed in this case.
