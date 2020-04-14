# Rust

- [Rust](#rust)
  - [Cargo](#cargo)
  - [Syntax/basics](#syntaxbasics)
    - [Basic structure/Printing/Input](#basic-structureprintinginput)
    - [Variables/Data types](#variablesdata-types)
    - [Closures](#closures)
    - [Ranges and `std::iter::Iterator` methods](#ranges-and-stditeriterator-methods)
    - [Arrays/Vectors](#arraysvectors)
    - [(String) Slices](#string-slices)
    - [For/while (/loop) loops](#forwhile-loop-loops)
    - [If/then/else](#ifthenelse)
    - [Pattern matching](#pattern-matching)
      - [Error handling](#error-handling)
    - [Basic operators/operations](#basic-operatorsoperations)
  - [Ownership](#ownership)
    - [Move](#move)
    - [Borrowing](#borrowing)
    - [Dangling pointers](#dangling-pointers)
    - [Slices](#slices)
  - [APIs](#apis)
    - [String-related](#string-related)
    - [Random](#random)

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
//
#[allow(dead_code)]
fn testing(n: u32) -> String {
  if n > 10 {
    panic("Error message!");
  }
  // In order to return a value without using `return`, omit the semicolon.
  //
  "abc"
}

fn main() {
  println!("Enter guess:");
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
  print!("Guess: {}", guess);
}
```

### Variables/Data types

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required
```

Integer types: `[iu](8|16|32|64|128|size)`. The `size` ones depend on the architecture.

All number literals support the type as suffix (e.g. `32u8`), and the (cosmetic) underscore.

Integer literals:

- hex: `0xff`,
- octal: `0o77`,
- binary: `0b1111_0000`,
- byte: `b'A'` (only `u8`; require single quotes).

Floats: `f(32|64)`.

Chars: `'🤯'` (4 bytes, require single quotes).

Tuples:

```rust
let (a, b, c) = (0, 1, 2);        // with multiple assignment
let first_element = tuple.0;      // tuple indexing
```

There are two types of strings:

- literals (`&'static str`); hardcoded in the executable.
- `std::string::String`s; allocated on the heap: `String::from("text")`.

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
rev()                        // reverse. WATCH OUT, UNINTUITIVE: since it's not inclusive, it goes from 99 to 0.
any(|x| x == 33)             // terminates on the first occurrence.
filter(|x| x == 33)          // iterator of the items verifying the condition; LAZY!
nth(n)                       // nth element (0-based)
take(n)                      // iterator for the first n elements
enumerate()                  // iterator (index, &value)
```

### Arrays/Vectors

Arrays (immutable, so they're allocated on the stack):

```rust
let my_list = [1, 2, 3];
let my_list = [true; 4]                 // 4 elements initialized as true; won't work with variable size (use a Vec)
let my_list[u32; 3] = [1, 2, 3]         // with data type annotation; ugly!
```

Vectors (mutable):

```rust
let mut vec = Vec::new();               // Basic instantiation
let mut vec = vec![1, 2, 3];            // Macro to initialize a vector from a literal list
let mut vec = vec![true; n];            // Same, with variable-specified length and initialization

vec.push(1);
vec.pop();

vec.len();
vec[0] = 2;

vec.extend([1, 2, 3].iter().copied());  // append a list
vec.extend(&[1, 2, 3]);                 // borrowing version
vec.iter()                              // iterator

vec.first();
vec.last();

println!("{:?}", vec);                  // must use this for printing
println!("{:#?}", vec);                 // pretty printing
```

### (String) Slices

Don't forget the `&` operator!!!

```rust
let string = String::from("pizza!");

// The type is `&str`
//
let s1 = &string[0..3];
let s1 = &string[..];                   // omitted start/end are syntax for start/end
```

String literals are String slices!

```rust
let string = String::new("My pizza");
let literal = "Your pizza";             // type is `&str` (string slice)

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

Slices apply to arrays as well:

```rust
let array = [8u32; 5];
let slice = &array[..];         // type is `&[u32]`
```

See the ownership chapter, for the related properties.

### For/while (/loop) loops

For loops iterate over ranges:

```rust
// For the reverse iteration, see the Ranges section, with warning(s).
//
for x in 0..10 { }

// Custom increment/decrement. See previous comment; goes from 98 to 0.
//
for x in (0..100).step_by(2).rev() {}

// Iterate an array.
//
for i in array.iter() { }
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
```

### Pattern matching

```rust
let xxx = match val {
  1 | 2 => "1 or 2",
  _     => {
    "other"
  },
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

### Basic operators/operations

```rust
val += 1; val -= 1;             // increment/decrement value (no postfix)
let val = 10_u64.pow(2);        // exponentiation (power)
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

### Slices

Slices are immutable references, so the ownership needs to be considered:

```rust
{
  let mut s = String::from("margherita");

  let slice = &s[..]; // Immutable borrow
  s.clear();          // Mutable borrow: Invalid!

  println!("{}", slice);
}
```

## APIs

### String-related

Conversions:

```rust
integer.to_string();                      // integer to string
let guess: u32 = string.parse().unwrap(); // string to numeric type
```

String APIs:

```rust
string.clear();                           // blank a string
string.len();
string.as_bytes();                        // byte slice of the string contents
```

Formatting:

```rust
format!("The number is {}", 1);                          // the template *must* be a literal (!)
format!("The number is {0}, again {0}, not {1}!", 1, 2); // numbered placeholders!
```

### Random

Requires `rand` dependency.

```rust
use rand::Rng;

// `thread_rng()`: seeded by the O/S; local to the current thread.
// `gen_range()`: close,open ends.
//
let secret_number = rand::thread_rng().gen_range(0, 2);
```