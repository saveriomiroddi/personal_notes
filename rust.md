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
    - [Arrays/Vectors](#arraysvectors)
    - [(String) Slices](#string-slices)
    - [For/while (/loop) loops](#forwhile-loop-loops)
    - [If/then/else](#ifthenelse)
    - [Pattern matching](#pattern-matching)
      - [Error handling](#error-handling)
    - [Structs](#structs)
    - [[Static] Methods](#static-methods)
  - [Ownership](#ownership)
    - [Move](#move)
    - [Borrowing](#borrowing)
    - [Dangling pointers](#dangling-pointers)
    - [Slices](#slices)
  - [APIs/Crates](#apiscrates)
    - [String/char-related](#stringchar-related)
    - [Random (`rand`)](#random-rand)
    - [Date/times (`chrono`)](#datetimes-chrono)

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
```

### Variables/Data types

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required

((1u128 << CONST_U64) - 1) as u64    // WATCH OUT the priorities! In this example, the brackets are all required!
```

Integer types: `[iu](8|16|32|64|128|size)`. The `size` ones depend on the architecture.

Max value function (example): `u64::max_value()`

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
let foo = ("bar", "baz");
let (bar, baz) = foo;             // with multiple assignment (unpacking); foo can also be a tuple literal
let first_element = tuple.0;      // tuple indexing

// Use a tuple as function argument
fn are(dimensions: (u32, u32)) -> u32 {
  dimensions.0 * dimensions.1
}
```

There are two types of strings:

- literals (`&'static str`); hardcoded in the executable.
- `std::string::String`s; allocated on the heap: `String::from("text")`.

### Basic operators/operations

```rust
val += 1; val -= 1;             // increment/decrement value (no postfix)
let val = 10_u64.pow(2);        // exponentiation (power)
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
filter(|x| x == 33)          // iterator of the items verifying the condition; LAZY!
nth(n)                       // nth element (0-based)
take(n)                      // iterator for the first n elements
enumerate()                  // iterator (index, &value)
join("str")                  // join using str
sum::<T>()

chunks(n)                    // iterate in chunks of n elements; includes last chunk, if smaller
chunks_exact(n)              // iterate in chunks of n elements; does not include the last chunk, if smaller
windows(n)                   // like chunks, but with overlapping slices

// transform an iterator into a collection
collect()
collect::<Vec<i32>>()
collect::<Vec<_>>()
```

### Arrays/Vectors

Arrays (immutable, so they're allocated on the stack):

```rust
let my_list = [1, 2, 3];
let my_list = [true; 4];                // 4 elements initialized as true; won't work with variable size (use a Vec)
let my_list: [u32; 3] = [1, 2, 3];      // with data type annotation; ugly!
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

vec.iter();                             // iterator
vec.extend([1, 2, 3].iter().copied());  // append a list
vec.extend(&[1, 2, 3]);                 // borrowing version

vec.first();
vec.last();
```

Arrays implement the `Debug` trait.

### (String) Slices

Don't forget the `&` operator!!!

```rust
let string = String::from("pizza!");
let string = "pizza!".to_string();

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

### String/char-related

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
string.push_str(&str);                    // concatenate (append) strings


// WATCH OUT! Does not split into graphemes, e.g. "ü" will be 2 chars (!); must 
// Must use a crate to handle this exactly.
//
string.chars();
```

Char APIs:

```rust
c.is_alphabetic();
c.is_numeric();
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

### Date/times (`chrono`)

```rust
// Don't use `time::Duration`, since it doesn't implement the operation traits.
//
use chrono::{DateTime, Duration, Utc};

let start:DateTime<Utc> = Utc::now();                   // Current time
start + Duration::days(3);                              // Arithmentic
start.checked_add(Duration::days(3));                   // Safe arithmetic
```
