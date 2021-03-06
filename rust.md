# Rust

- [Rust](#rust)
  - [Cargo/Rustup](#cargorustup)
    - [Features](#features)
    - [Toolchain](#toolchain)
    - [Fast builds](#fast-builds)
    - [Cross-compilation](#cross-compilation)
  - [Rustfmt](#rustfmt)
  - [Syntax/basics](#syntaxbasics)
    - [Basic structure/Printing/Input](#basic-structureprintinginput)
      - [Printing/formatting/write!](#printingformattingwrite)
    - [Conditional build (ifdef-like)](#conditional-build-ifdef-like)
    - [Data types](#data-types)
      - [Numeric operations](#numeric-operations)
    - [Casting](#casting)
      - [Into<T>](#intot)
    - [Basic operators/operations/arithmetic/math](#basic-operatorsoperationsarithmeticmath)
    - [Closures/Functions](#closuresfunctions)
    - [Ranges and `std::iter::Iterator` methods](#ranges-and-stditeriterator-methods)
      - [Method chaining](#method-chaining)
      - [Iterator trait/Associated types](#iterator-traitassociated-types)
    - [Arrays/Vectors/Slices](#arraysvectorsslices)
    - [Hash maps](#hash-maps)
    - [Strings](#strings)
      - [String/char-related APIs/conversions](#stringchar-related-apisconversions)
      - [Internal representation (bytes/chars/graphemes)](#internal-representation-bytescharsgraphemes)
    - [For/while (/loop) loops](#forwhile-loop-loops)
    - [If (let)/then/else](#if-letthenelse)
    - [Sorting](#sorting)
      - [Sorting floats](#sorting-floats)
    - [Enums](#enums)
    - [Option<T>/Result<T, E>](#optiontresultt-e)
    - [Pattern matching](#pattern-matching)
      - [Error handling](#error-handling)
    - [Structs](#structs)
    - [`self` in methods](#self-in-methods)
    - [Generics](#generics)
    - [Traits](#traits)
      - [Basics (and Generics #2)](#basics-and-generics-2)
      - [Approaches to collections and static/dynamic dispatching](#approaches-to-collections-and-staticdynamic-dispatching)
      - [Supertraits and inheritance (object-orientation)](#supertraits-and-inheritance-object-orientation)
        - [Inheritance: making overridden methods private](#inheritance-making-overridden-methods-private)
      - [Limitations of returning Self](#limitations-of-returning-self)
      - [Downcasting/Upcasting](#downcastingupcasting)
    - [Operator overloading](#operator-overloading)
    - [Method overloading (workaround)](#method-overloading-workaround)
    - [Associated functions/methods](#associated-functionsmethods)
    - [Ownership](#ownership)
      - [Move](#move)
      - [Borrowing](#borrowing)
      - [Dangling pointers](#dangling-pointers)
      - [Lifetimes](#lifetimes)
      - [Slices](#slices)
    - [Smart pointers](#smart-pointers)
      - [Box<T>](#boxt)
      - [RC<T>](#rct)
      - [RefCell<T>/Mutex<T> and interior mutability](#refcelltmutext-and-interior-mutability)
      - [Modifying Rc/Arc without inner mutable types (and conversion Box -> RC type)](#modifying-rcarc-without-inner-mutable-types-and-conversion-box---rc-type)
      - [Weak<T> and reference cycles](#weakt-and-reference-cycles)
        - [`Rc<RefCell>` or `RefCell<Rc>`](#rcrefcell-or-refcellrc)
        - [Real case of modeling a thread-safe tree with trait objects, with children addition](#real-case-of-modeling-a-thread-safe-tree-with-trait-objects-with-children-addition)
        - [Real complex case of iterating a recursive structure with Rc/RefCell](#real-complex-case-of-iterating-a-recursive-structure-with-rcrefcell)
    - [Multithreading](#multithreading)
      - [Channels: Multiple Producers Single Consumer](#channels-multiple-producers-single-consumer)
      - [Mutex<T>/Arc<T>](#mutextarct)
      - [Atomic primitive type wrappers](#atomic-primitive-type-wrappers)
      - [Barrier](#barrier)
      - [Condvar](#condvar)
      - [Busy waiting/spin loops (pause)](#busy-waitingspin-loops-pause)
    - [Unsafe](#unsafe)
      - [Interoperability with other languages (C)](#interoperability-with-other-languages-c)
    - [Macros](#macros)
      - [Function-like](#function-like)
        - [Rules and details](#rules-and-details)
        - [Importing](#importing)
      - [Custom derive macros](#custom-derive-macros)
    - [Unions](#unions)
    - [Assorted insanity](#assorted-insanity)
  - [Packaging](#packaging)
    - [Project structure](#project-structure)
    - [Modules](#modules)
  - [Traits](#traits-1)
    - [Default](#default)
    - [Copy, Clone, Drop and their relationships](#copy-clone-drop-and-their-relationships)
    - [Display](#display)
    - [From (/Into)](#from-into)
    - [Index[Mut]](#indexmut)

## Cargo/Rustup

Base operations:

```sh
cargo new "$project_name" [--lib]
cargo check                          # check for errors
cargo run
cargo build [--release]              # builds (default: debug); if necessary, updates the crates index, and installs the dependencies
cargo update                         # updates the crates index, and the installed dependencies
cargo test [-- --nocapture]          # pass nocapture in order to output print statements
cargo fmt
cargo clippy                         # linter
cargo doc [--open]                   # builds and optionally opens docs for the installed crates

# Build/run options

--features draw_hitboxes
```

When a project is run via Cargo, the env variable `CARGO_MANIFEST_DIR` is passed to the binary.

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

[[test]]
harness = false # Allow Cucumber to print output instead of libtest
name = "cucumber"

[dev-dependencies]
cucumber = {package = "cucumber_rust", version = "^0.7.0"}

[dependencies]
rand = "0.7.3"
redisish = { path = "../redisish" }                         # Relative dependency
amethyst = { git = "https://github.com/amethyst/amethyst" } # Repository; options: `branch`/`tag`/`rev` (master branch is the default)

# Another way to declare a dependency
[dependencies.amethyst]
features = ["vulkan"]
version = "0.15"

# There are 4 profiles: `dev`, `release`, `test`, and `bench`
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
```

Versioning is pessimistic by default.

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.

The cargo configuration file (see custom configs below) `toml` extension is optional; it's search in many locations: https://doc.rust-lang.org/cargo/reference/config.html.

### Features

Define the features of a project.

```toml
[features]
default = ["draw_hitboxes"] // features enabled by default
draw_hitboxes = []          // `[]` is the dependencies
```

### Toolchain

Set the specified toolchain for the current project:

```sh
# Using rustup; this is stored in rustup's config.
#
rustup override set nightly
rustup override unset # raise error if none is set

# Explicit alternative
#
echo nightly > rust-toolchain
```

### Fast builds

Bevy hello world: 8.75" -> 1.25" (!!).

```sh
# If using rust stable, remove the "-Zshare-generics=y" below.
# Requires lld. Source: git.io/JsfhD (includes other O/Ss).

mkdir .cargo
cat > .cargo/config.toml << 'TOML'
[target.x86_64-unknown-linux-gnu]
linker = "/usr/bin/clang"
rustflags = ["-Clink-arg=-fuse-ld=lld", "-Zshare-generics=y"]
TOML
```

### Cross-compilation

```sh
# Preparation. Requires Ubuntu package `gcc-mingw-w64-x86-64`
#
rustup target add x86_64-pc-windows-gnu
rustup toolchain install stable-x86_64-pc-windows-gnu

# Cross-compile
#
cargo build --release --target x86_64-pc-windows-gnu

# Alternative: Configure Cargo for cross-compilation:
#
mkdir .cargo
cat > .cargo/config.toml <<TOML
[build]

target = "riscv64gc-unknown-linux-gnu"

[target.riscv64gc-unknown-linux-gnu]

linker = "riscv64-unknown-linux-gnu-gcc"
TOML
```

## Rustfmt

Use the attribute macro `#[rustfmt::skip]` in order to disable automatic formatting.

Add a `rustfmt.toml` to the project root. See: https://rust-lang.github.io/rustfmt (don't forget to consider the channel).

```toml
# Unstable features work only on nightly!
#
unstable_features = true

blank_lines_upper_bound = 10             # unstable
preserve_block_start_blank_lines = true  # unstable
```

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
fn main() -> Result<(), Box<dyn Error>> {
  print!("Enter guess: ");
  io::stdout().flush().unwrap(); // makes sure that the output is flushed, since O/S generally do it per-line.

  // `mut`: mutable.
  // the `new` function is not dictated by the language, but a common practice.
  //
  let mut buffer = String::new();

  // Appends to the buffer.
  //
  let bytes_read = io::stdin().read_line(&mut buffer)?;

  // Placeholder: `{}`
  //
  println!("Guess: {}", guess);

  // See fn return value.
  //
  Ok(());
}
```

"Divergent" functions are functions that never return, marked with `!` return type; exiting from them is considered an error:

```rust
fn cycle_forever() -> ! {
  loop { /* ... */ }
}
```

#### Printing/formatting/write!

WATCH OUT! Don't forget to bring `std::io::Write` into scope, when using write-related traits.

```rust
println!("{:#?}", vec);                 // generic pretty printing
println!("{:?}", vec);                  // `Debug` format (requires the `Debug` trait; if generic, requires `use std::fmt::Debug`)

eprintln!("Error!");                    // print on stderr!

format!("The number is {}", 1);                          // the template *must* be a literal (!)
format!("The number is {0}, again {0}, not {1}!", 1, 2); // numbered placeholders!

writeln!(writer, "{}", 123).unwrap();   // write formatted data into a Write implementor; `write()` also available
```

Formatting (see https://doc.rust-lang.org/std/fmt):

```rust
"{:.2}"             // round float
"{:x}/{:X}"         // lower/upper hex
"{:b}"              // binary (via trait std::fmt::Binary)
"{:p}"              // pointer

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

Sample Debug implementation:

```rust
impl Debug for SortableFloat {
  fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
    write!(f, "{}", self.0)
  }
}
```

### Conditional build (ifdef-like)

The `cfg` attribute performs a conditional build:

```rust
#[cfg(test)]                 // Compile only in test builds
#[cfg(not(test))]            // ... in non-test builds
#[cfg(target_os = "linux")]  // ... on linux
#[cfg(feature = "draw_hitboxes")] // ... with feature enabled
```

The attribute can be applied to methods, statements, etc.

### Data types

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
let bytestr: &[u8] = b"hello";      // byte string
let rawstr: &str = r"hello";        // raw string (doesn't process escapes)
let byterawstr: &[u8] = br"hello";  // byte raw string

// Any number of hashes can be added to the raw string delimiters, to disambiguate the ending one:

let zzz = br#"foo"bar"#;
let zzz = br##"foo"#bar"##;         // with a single delimiting hash, the first `"#` is considered terminating
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

#### Numeric operations

```rs
5_u64.wrapping_add(-4_i32 as u64);   // proper way to add signed to unsigned
5_u64.checked_sub(-(-4_i32) as u64); // otherwise, branch positive/negative, and do checked op for both cases
```

### Casting

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required

((1u128 << CONST_U64) - 1) as u64   // WATCH OUT the priorities! In this example, the brackets are all required!

type Kilometers = i32;                     // type aliasing
type Result<T> = Result<T, std::io:Error>; // library example: `std::io::Result`

let bool_as_int = true as i32;      // true: 1, false: 0
let int_as_bool = 1 as bool;        // 1: true, 0: false, other: !!undefined!!
```

For static variables, see [static section](#staticglobal-variables-lazy_static-once_cell-thread_local)

Number-related casts/operations:

```rust
0xFF_u8 as u16;          // 0x00FF ("zero-extend")
  -1_i8 as u16;          // 0xFFFF ("signed-extend")

0xFF_u8 as i16;          // WATCH OUT!!: 0x00FF
(0xFF_u8 as i8) as i16;  // 0xFFFF

1.to_string();             // numeric to string

integer.to_string();                      // integer to string
String::from_utf8(bytes).unwrap();        // (valid) utf-8 bytes to string
char.to_digit(RADIX).unwrap();            // (parse) char to numeric

// parse string to numeric type; with any numeric implementing `FromString`
// f64 will parse integer strings (e.g. `1`)
//
let guess: u32 = string.parse().unwrap();
```

Automatic casting, for types supporting the `Deref` trait:

- `&String` -> `&str`
- `&Vec<T>` -> `&[T]`
- `&Box<T>` -> `&T`

#### Into<T>

In order to implement arbitrary casting, implement the `Into<T>` trait:

```rust
// Allows `GamePiece { color: BLACK } as i32`
//
impl Into<i32> for GamePiece {
  fn into(self) -> i32 {
    if self.color == BLACK { 0 } else { 1 }
  }
}
```

### Basic operators/operations/arithmetic/math

WATCH OUT!! `-1.operation(2)` is interpreted as `-(1.operation(2))`!!

```rust
x / y                           // nearest int (-3 / 2 == -1) -> different from Ruby
val += 1; val -= 1;             // increment/decrement value (no postfix)
val <<= n; val >>= n;           // overflows are ignored
std::mem::swap(&mut a, &mut b); // !! swap two variables !!
std::mem::size_of::(Type)       // memory occupation of a type
std::mem::size_of_val(v)        // memory occupation of a variable !!

10_f64.clamp(-100., 100.)       // clamp a number (limit value within interval); floats only
10_u64.pow(2);                  // exponentiation (power), int/int
10_f64.powi(2);                 // exponentiation, float/int
10_f64.sqrt();                  // square root
(-1_f64).rem_euclid(20);        // 19 (nonnegative remainder); -1 % 20 == -1; see https://doc.rust-lang.org/std/primitive.i32.html#method.rem_euclid
10_f64.sin();                   // sine (in rad)
10_f64.cos();                   // cosine (in rad)
10_f64.signum();                // sign. float: (>= +0.0 -> 1.0), (<= -0.0 -> -1.0), (NaN -> NaN); int: (0 -> 0), (< 0 -> -1), (> 0 -> 1)
10_f64.hypot(10_f64);           // hypotenuse (!!)

std::f64::consts::PI;
std::f64::INFINITY, NEG_INFINITY;

u32::min(1, 2);                 // minimum between two numbers
std::cmp::max(x, u);            // maximum between two numbers

z, carry = x.overflowing_add(y); // adds and wraps around in case of overflow; <carry> is bool.
z, carry = x.overflowing_sub(y); // subtracts and wraps around, as above
                                 // WATCH OUT! For other `overflow_` operations, `carry` may not the intuitive value, eg. for bit shift
z = x.rotate_left(y);
z = x << y;                      // Shift; errors only if y is higher than the number of machine bits.
(_, _) = x.overflowing_shl(y);   // WATCH OUT! This is not the intuitive 1-bit left shift - Rust doesn't have it predefined; for that,
                                 // manually compute the carry, then execute `wrapping_shl()`. Alternatively, override the shift operator.

(f * 100.0).round() / 100.0;     // round to specific number of decimals (ugly!!; also see #printing)
0_u32.to_be_bytes();             // convert big endian u32 to array of bytes
u32::from_le_bytes([u8; _])                   // convert big endian array of bytes to u32
u32::from_le_bytes(&[u8].try_into().unwrap()) // same, from slice; requires `std::convert::TryInto`
f64.to_bits()                    // transmute to u64 (for bit-wise ops)
f64::from_bits(u64)              // transmute from u64 (for bit-wise ops)
```

### Closures/Functions

Equivalent of Ruby blocks!

```rust
let multiple_of_10 = |x| { x % 10 == 0; }   // yay! note: the braces are optional
(0..100).any(multiple_of_10);               // double yay!
let is_0 = |x: i32| -> bool { x == 0; }     // with type annotations; they're not required

// Generic closure signature. The closure types passed don't need to be annotated.
//
struct Calculator<T: Fn(u32) -> u32> {
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

If the error `expected fn pointer, found fn item` is raised, explictly define the fn type (seems related to git.io/JGz2L).

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

`const` closures [can't be defined](https://stackoverflow.com/questions/29191170/is-there-any-way-to-explicitly-write-the-type-of-a-closure).

### Ranges and `std::iter::Iterator` methods

General form: `[start] .. [[=]end]`.

Ranges are:

- lazy;
- open ended on the `end`, unless `=` is specified.

Iterator getting methods:

```rust
collection.iter()            // immutable references
collection.iter_mut()        // mutable references
collection.into_iter()       // owned values (arrays still return references)
```

`std::iter::Iterator` methods, implemented by Range:

```rust
map(|x| x * 2)               // Ruby :map
map(|(x, y)| x + y)          // Tuples unpacking: useful for example, on the result of zip()
flat_map(|x| x)              // Ruby :flat_map. WATCH OUT! flattens only one level.
fold(a, |a, x| a + x)        // Ruby :inject
fold_first(|a, x| a + x)     // Like fold(), using the first element as initial value
filter(|x| x % 2 == 0)       // Ruby :select
filter_map(|x| Some(x * 2))  // AWESOME!!! Combines filter and map; None values are discarded
find(|x| x % 2 == 0)         // Ruby :find/:detect
flatten()                    // Quasi-Ruby :flatten. WATCH OUT! flattens only one level.
dedup()                      // Ruby :uniq
rev()                        // reverse. WATCH OUT, UNINTUITIVE: since it's not inclusive, it goes from 99 to 0.
any(|x| x == 33)             // terminates on the first true
all(|x| x % 2 == 0)          // terminates on the first false
nth(n)                       // nth element (0-based)
skip(n)                      // skip n elements
take(n)                      // iterator for the first n elements
enumerate()                  // iterator (index, &value) (Ruby :each_with_index)
join("str")                  // join using str; doesn't join `char` collections
zip(iter)                    // zip two arrays (iterators)!!!
sum()                        // WATCH OUT! Returns the same type, so conversion is needed, e.g. `.map(|&x| x as u32).sum();`

// Can't unpack directly in the `for` with these, since they're refutable patterns.
//
chunks(n)                    // iterate in chunks of n elements; includes last chunk, if smaller
chunks_exact(n)              // (preferred) iterate in chunks of n elements; does not include the last chunk, if smaller
windows(n)                   // like chunks, but with overlapping slices (Ruby :each_cons)

// If one wants to convert an iterator from borrowed (&T) to owned (T), use copied() or cloned().
// Example, from [f64; _] to Vec<f64>:
myarr_f64.iter().flatten().copied().collect::<Vec<_>>();
myarr_f64.iter().flatten().cloned().collect::<Vec<_>>();

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
let arr = [1, 2, 3];
let arr = [true; 4];                     // 4 elements initialized as true; won't work with variable size (use a Vec)
let arr: [u32; 3] = [1, 2, 3];           // with data type annotation; ugly!
let mut arr: [Option<u32>; 3] = [None; 3];  // with Option<T>; super-ugly!

// In order to store mixed types, use a tuple. Example of pseudo-bidimensional array:
//
let mtarr = [
  ((1.0, 2.0), -1, "cde"),
  // ...
];

// If a struct has an array, it's possible to initialize the array implicitly by using the [Default](#default) trait on the struct.

arr[512..512 + source.len()].copy_from_slice(&source) // memcpy (copy) from/to slices/vectors; source/dest size must be the same!
arr.fill(value)                         // memset; unstable as of Aug/2020
[1, 2, 3] == [1, 2, 4]                  // arrays can be compared via `==`

arr.to_vec()                            // convert array to vector

// invocation: process_list(&my_list)
//
fn process_list(list: &[i32]) {}

// For unpacking, see `if let`
```

Watch out!! Very large arrays will overflow the stack; use `Vec` in such cases.

Vectors (mutable):

```rust
let mut vec = Vec::new();               // Basic (untyped) instantiation (if the type can be inferred)
let mut vec: Vec<i32> = Vec::new();     // Basic, if type can't be inferred
let mut vec = vec![1, 2, 3];            // Macro to initialize a vector from a literal list; allocates exact capacity
let mut vec = vec![true; n];            // Same, with variable-specified length and initialization; ; allocates exact capacity
Vec::with_capacity(cap);                // Preallocating version; WATCH OUT! The length is still 0 at start; use `vec![<val>, n]` if required

vec[0] = 2;

let val = &vec[0];
vec.get(2);                             // Safe (Option<T>) version
vec.first();
vec.last();

vec.push(1);                            // Push at the end
vec.pop();                              // Pop from the end
let old = vec.swap_remove(i);           // Fast removal (fills in with the last element)
std::mem::swap(&mut new, &mut vec[0])   // Replace an entry with another

vec.extend([1, 2, 3].iter().copied());  // Append (concatenate) an array
vec.extend(&[1, 2, 3]);                 // Append (borrowing version)
vec[range].copy_from_slice(&source);    // memcpy; see array example

// Convert vector to array; requires TryInto. If type annotations are required, use the second form:
//
vec.as_slice().try_into().unwrap();
TryInto::<&[f64; 3]>::try_into(vec.as_slice());

// Vectors can be received as array reference type (mutable, if required).
//
fn process_list<T>(list: &[T]) {};
process_list(&vec);
```

In order to unpack a vector, convert to slice and use if let (see [pattern matching](#pattern-matching)).

Element(s) conditional removal:

```rust
// Remove one match; ignore if element not found.
//
if let Some(i) = vec.iter().position(|vec_item| *vec_item == remove_item) {
  vec.remove(i);
}

// Remove all matching elements (Ruby delete_if(), but with opposite condition)
//
vec.retain(|vec_item| *vec_item != remove_item);
```

Common methods:

```rust
vec.swap(pos1, pos2);
vec.resize(new_len, value: T);          // resize (extend/shrink) a vector; T must be Clone
vec.resize_with(new_len, || expr);      // resize, via function (e.g. when T is not Clone)

// Splitting
(sl1, sl2) = coll.split_at(split_point);     // immutable
(sl1, sl2) = coll.split_at_mut(split_point); // mutable: the two arrays are subarrays of the source
coll2 = coll.split_off(split_point);         // mutable: the returned (sub)array is removed from the source

vec.len();
vec.iter();                             // iterator

// Sorting (destructive!)
vec.sort();                             // stable
vec.sort_by_key(|e| e.abs());           // stable, by key!
vec.sort_unstable();                    // unstable (typically faster than stable)
vec.sort_by(|a, b| a.partial_cmp(b).unwrap()); // sort with closure 😍!
```

In order to pick a random element, see [rand crate](#random-with-and-without-rand).

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

The default hashing function is cryptographically secure!!. For faster versions, must use a crate (common one: `fxhash`). For a version that can be used as `const`, see [`phf`](rust_ruby_libraries.md#)

WATCH OUT! In order to avoid BCK headaches when getting and setting in the same scope/function, use the [Entry API](https://stackoverflow.com/a/28512504/210029) (see below).

```rust
use std::collections::HashMap;

// When choosing between owned/borrowed, keeping in mind that if elements are created in a function
// and returned, ownership is necessary.
//
let mut map = HashMap::new();

map.insert("b", 10);
map.insert("b", 10);            // Overwrites the existing value

// Entry API: `entry()` gets the value for in-place modification.
//
// `or_insert()` sets the given value if the key doesn't exist; its return value can be used to modify
// the value in-place; consumes the entry.
//
// If the value to insert is a function (or relatively expensive), use `or_insert_with(Fn)`, which executes
// only if the value is going to be inserted.
//
let value_ref: &u32 = map.entry(key).or_insert(50);
*value_ref = 100;
map.entry(key).or_insert_with(Vec::new);

// Modify a map in a loop.
//
for triangle in triangles {
  let group = groups.entry(current_group);
  group.and_modify(|group| Group::add_child(group, &triangle));
}

// WATCH OUT!: Getters take and return references.
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

// Iterate a map
//
for (book, review) in &book_reviews { /* ... */ };

// Other APIs
//
map.keys();    // Iterator
map.values();  // Iterator

// Keys may need a bit of treatment in order to be collected. For example, if the map has String keys,
// the &String references will need to be converted to slices:
//
map
  .keys()
  .map(|k| k.as_str())
  .collect::<Vec<_>>()
  .as_slice();
```

Conveniences:

```rust
// Convert a map to owned vector
//
let vec = score_table
  .into_iter()
  .map(|(_k, v)| v)
  .collect::<Vec<_>>();

// Create a map from two arrays.
//
let scores = teams
  .iter()
  .zip(initial_scores.iter())
  .collect::<HashMap<_, _>>();

// Create a map from a vector of tuples (a, b)
//
let haxx: HashMap<_, _> = vec![("foo", 1), ("barry", 2)]
    .into_iter()
    .collect();
```

In order to use enums as keys, annotate them with `#[derive(Eq, PartialEq, Hash)]`.

Map literals are not supported. See the `maplit` crate.

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

// multi-line string, with leading spaces removed.
// for the Ruby squiggly heredoc, use the `indoc` crate.
//
let string = "\
  a\n\
    b c\n\
    d \n\
"
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

#### String/char-related APIs/conversions

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
s.insert(pos, 'c');                     // insert; use also as Ruby unshift()
s.replace_range(range, "s");            // replace a string range (!)
s.to_lowercase(); s.to_uppercase();
s.replace("a", "b");                    // gsub
s.clear();                              // blank a string
s.repeat(8);                            // string repeat (multiplication)

s.trim(); s.trim_end(); s.trim_start(); // trim/strip
s.trim_end_matches("suffix");           // chomp suffix (but repeated)! also accepts a closure

s.as_bytes();                           // byte slice (&[u8]) of the string contents
s.into_bytes();                         // convert to Vec[u8]

// splits; there is a `rsplit*` version for each
// in order to use the slice methods, do `collect()`
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
// The `external` lifetime is an (optional) example.
//
'external
loop {
  if true {
    break 'external;    // can return a value
  }
};
```

### If (let)/then/else

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

// In order to compose a complex expression with pattern matching, use `matches!`
//
if matches!(coin, Coin::Quarter(x) if x > 2) && random_event() {
  // Can use a guard (above), but can't use the value here (makes sense)!
}
```

### Sorting

Definitions:

- `PartialEq`: the type is a partial equality relation (floats isn't, because NaN != NaN); allows comparison with asserts.
- `Eq`: the type is an equality relation; good practice to implement if applies.

Watch out! The f64 doesn't support Ord (which is often required); only PartialOrd.

#### Sorting floats

This is the generic solution, supporting NaN. A collection without NaN can rely on `a.partial_cmp(b).unwrap()`.

Required a wrapper class (and a truckload of boilerplate):

```rust
// Implementation that considers NaN as greater than the other floats, and equal to itself.
// This doesn't conform to the standard.
//
pub struct SortableFloat(pub f64);

// This only informs the compiler that the type supports (full) equivalence.
//
impl Eq for SortableFloat {}

impl PartialEq for SortableFloat {
  fn eq(&self, other: &Self) -> bool {
    if self.0.is_nan() {
      other.0.is_nan()
    } else {
      self.0 == other.0
    }
  }
}

impl Ord for SortableFloat {
  fn cmp(&self, other: &Self) -> Ordering {
    let (lhs, rhs) = (self.0, other.0);

    if let Some(result) = lhs.partial_cmp(&rhs) {
      result
    } else {
      if lhs.is_nan() {
        if rhs.is_nan() {
          Ordering::Equal
        } else {
          Ordering::Greater
        }
      } else {
        Ordering::Less
      }
    }
  }
}

impl PartialOrd for SortableFloat {
  fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
    Some(self.cmp(other))
  }
}
```

### Enums

Definition:

```rust
// Each entry is called "variant".
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

// Enums can be associated to values.
// `repr(u8)` forces the compiler to use 8-bits for each variant.
//
#[repr(u8)]
enum Color {
    WHITE = 0x00,
    BLACK = 0x01,
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

Enums can't be iterated. See `strum` crate for this purpose.

### Option<T>/Result<T, E>

Foundation of Rust. In order to use the contained value, we must extract (and test) it.

Many of the methods are common between `Option` and `Result`.

WATCH OUT! unwrap() moves the value out, so trying to do `&mut opt.unwrap()` doesn't work, because it's a reference to the result, rather than the content.  
In order to solve this case, use `if let` or `as_ref()`/`as_mut()`.

```rust
enum Option<T> {
  Some(T),
  None,
}

// For None, the type must be specified, otherwise it can't be inferred.
//
let some_number = Some(5);
let absent_number: Option<i32> = None;


// !!!!!! WATCH OUT! The Result error enum is `Err`, not `Error` !!!!!!

// Question mark ('?') operator: convenient syntax for returning None/Err from the function, if it's the value of an Option/Result.
// If the Error type is different and From<T> is implemented, it's invoked.
//
fn mine() -> Result<String, io::Error> {
  let value = errorable_operation()?;
  result = process(value);
  Ok(result)
}
//
// Desugared version.
//
let result = match Try::into_result(expression) {
    Ok(v) => v,
    Err(e) => return Try::from_error(From::from(e)),
};


// Convenient pattern. Companion APIs:
//
// - `unwrap_or`:         eagerly evaluated
// - `unwrap_or_default`: invokes the `default()` (!!)
//
opt.unwrap_or_else( |err | {
  println!("Problem parsing arguments: {}", err);
  std::process::exit(1);
});

// Chain operations, stopping at None.
//
opt.and(value).and_then(|| slow_function());

// Extract a value, raising an error if None. Since `expect()` consumes the value, use `as_ref()`/
// `as_mut()` in order to borrow.
//
let value = opt.expect("it shouldn't be None!");
let &mut value = opt.as_mut().expect("it shouldn't be None!");

// Convert Result to Option, and discard the error (use this if it's sure that there can't be an error)
//
result.ok();

// Convert Option to Result.
//
option.ok_or("error!");
option.ok_or_else(|| slow_function() );

// take(): extract a value and replace with None (no errors raised if invoked on None).
// This is useful when we want to move out an instance that doesn't implement Copy.
//
let mut x = Some(2);
let y = x.take();
x == None;
y == Some(2);

// replace(): extract a value and replace it with the value passed.
//
let mut x = Some(2);    // works also on None
let old = x.replace(5);

// map(): map the content, only if it's Some.
//
let (with, without) = (Some(2), None::<i32>);
with.map(|n| 2 * n); // Some(4)
without.map(|n| 2 * n); // None

// Mappings borrowed <> owned
//
Some(12).as_ref()
Some(&12).cloned()
```

See [pattern matching](#pattern-matching).

### Pattern matching

`let` can be used to destructure (e.g. a struct) even without being an `if let`!

(for testing a value directly, one can use `value.is_none()`/`value.is_some()`).

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

// Convenient macro.
// When implementing on PartialCmp, import `std::cmp::Ordering::{Equal, Greater, Less}`.
//
matches!(self.partial_cmp(other), Some(Less) | Some(Equal))

// Match enum; all the entries must be exhausted (or the `_` placeholder must be used).
// The second case is a variant with an associated value!
//
// See `if let` for an alternative structure.
//
match message {
  Message::Move{x, y} => { self.position = Point { x: x, y: y } }
  Message::Echo(s) => { self.echo(s) }
  Message::ChangeColor(c1, c2, c3) => { self.color = (c1, c2, c3) } // enum of tuple struct
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

// Match (unpack) slices/arrays

if let [r, g, b] = &raw_pixels[..] { /* .. */ } else { panic!() }

// Match Option<T>
//
let x = Some(10);
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
let User {username: a, active: b, ..} = user;
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

### `self` in methods

In methods, the `self` reference can be also a smart pointer:

```rust
struct MyStruct {
  fn my_method(self: Arc<Self>, param: u32) { /* .. */ }
}
arc_ref = Arc::new(MyStruct {});
arc_ref.my_method(123);
```

This is currently limited to `Box`/`Rc`/`Arc`.

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

### Traits

#### Basics (and Generics #2)

```rust
pub trait Summary {
  // Trait constants are not constants in a strict sense - they're defaults. They must be accessed from the implementing type (in this
  // case, Article::MAX_LENGTH or <Article as Summary>::MAX_LENGTH).
  //
  const MAX_LENGTH: u16 = 4096;

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

// Traits can be implemented only if the trait or the type are local to the crate!
// This way, one doesn't find "surprises" from other crates.
//
impl Summary for Article {
  fn summarize(&self) -> String {
    // See note above on trait constants.
    //
    if text.len() > Self::MAX_LENGTH {
      panic!();
    }

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

#### Approaches to collections and static/dynamic dispatching

Let's say we have the following trait + concrete types:

```rust
pub trait Draw { fn draw(&self); }

pub struct Square {}
impl Draw for Square { fn draw(&self) {} }

pub struct Circle {}
impl Draw for Circle { fn draw(&self) {} }
```

If we want a generic collection of instances of the same type, we use generics; this is faster, as it's static dispatching:

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

// Syntax for defining methods with generics.
impl<T: Draw> Screen<T> { pub fn run(&self) {} }

let screen = Screen { components: vec![Square {}, Square {}] };
```

If the instances can be of different types, we use dynamic dispatching:

```rust
pub struct Screen {
  // In the past, this could be defined as `Vec<&Draw>`, which is now deprecated.
  pub components: Vec<Box<dyn Draw>>,
}
impl Screen { pub fn run(&self) {} }

let screen = Screen { components: vec![Square {}, Circle {}] };
```

`dyn <type>`s are called "trait objects". They require a pointer, like `Box<T>` or a reference (`&`); the methods in the thread must be "object safe":

- return type isn't self;
- no generics.

In order to specify a dynamic type when boxing, must explicitly cast:

```rust
Box::new(NullLogger {});                    // type = Box<NullLogger>
Box::new(NullLogger {}) as Box<dyn Logger>; // type = Box<dyn Logger>
```

#### Supertraits and inheritance (object-orientation)

Supertraits are traits depending on other traits. `X` is a supertrait of `Y` means that `Y` must implement `X` (this "super" as higher in a tree).

```rust
// Example using default implementation.
//
trait BetterDisplay: fmt::Display {
    fn better_to_string(&self) -> String {
        format!("Better!: {}", self.to_string())
    }
}

// - Multiple supertraits;
// - Generic supertrait returning self -> requires Sized
//
trait Matrix: Sized + Mul<Self> { /* ... */ }
```

We can use supertraits in order to implemented object-oriented inheritance; the essence is that the tree is based on traits, rather than types:

```rust
trait Root {
    fn update(&self) {
        println!("M:root");
    }

    fn type_() {
        println!("AF:root");
    }
}

trait Middle: Root {
    fn update(&self) {
        Root::update(self);
        println!("M:middle");
    }
}

trait Leaf: Middle {
    fn update(&self) {
        Middle::update(self);
        println!("M:leaf");
    }
}

struct Node {}

// Note that we need to implement all the traits.
impl Root for Node {
    fn type_() {
        println!("AF:node+root");
    }
}
impl Middle for Node {}
impl Leaf for Node {}

impl Node {
    pub fn update(&self) {
        println!("M:node");
    }

    pub fn type_() {
        println!("AF:node");
    }
}

let node = Node {};

// Note the so-called disambiguations (#1 and #4/5)

Leaf::update(&node);      // M:root/M:middle/M:leaf
node.update();            // M:node                 - if `impl` Node is not provided, this is invalid

Node::type_();            // AF:node
<Node as Root>::type_();  // AF:node+root           - if `impl` Node is provided
<Node as Root>::type_();  // AF:root                - if `impl` Node is not provided
Root::type_();            // Invalid!               - must invoke on an `Impl` type; traits alone are not objects!
```

##### Inheritance: making overridden methods private

The simplest way to make overridden methods private is to use private trait methods:

```rust
// #![feature(get_mut_unchecked)]
// #![feature(test)]
//
/////////////////////////////////////
//
// #![allow(unused_must_use)]
// #![allow(dead_code)]
#![allow(unused_variables)]
#![allow(unused_imports)]
// #![allow(unused_assignments)]
// #![allow(unused_macros)]
// #![allow(unused_mut)]
// #![allow(non_snake_case)]

pub(crate) mod private {
    pub trait SubInterface {
        fn redefined(&self);
    }
}

pub trait SuperClass: private::SubInterface {
    fn virtual_(&self) {
        println!("virtual_");
        self.redefined()
    }
}

struct SubClass {}

impl private::SubInterface for SubClass {
    fn redefined(&self) {
        println!("redefined");
    }
}

impl SuperClass for SubClass {
    fn virtual_(&self) {
        println!("ovr-virtual_");
        private::SubInterface::redefined(self);
    }
}

let instance = SubClass {};

instance.virtual_(); // virtual_/redefined     - if virtual_() not overridden
instance.virtual_(); // ovr-virtual_/redefined - if virtual_() is overridden
```

An ugly alternative is to pass a "sub interface" implementors a "base class" type:

```rust
impl BaseClass {
    fn virtual_<T: SubInterface>(&self, redefiner: T) {
        redefiner.redefined();
    }
}
```

The other strategy is embedding; in order to avoid forwarding boilerplate, there is the [`delegate` crate](#inheritance-emulation-via-delegate-crate), or the [`Deref\[Mut\]` antipattern](https://github.com/rust-unofficial/patterns/blob/master/anti_patterns/deref.md)).

#### Limitations of returning Self

See https://stackoverflow.com/q/30938499.

```rust
trait MyTrait {
  // This can return Self, which is ?Sized (not necessarily sized), because the Sized responsibility
  // is down to the implementor.
  //
  fn new() -> Self;

  // In this case, the responsibility in the trait, therefore, Self needs to be Sized.
  // either add `where Self: Sized` to the method, or add the `Sized` bound to the trait.
  //
  fn default_new() -> Self {
    Self::new()
  }
}
```

#### Downcasting/Upcasting

Downcasting from trait object to the concrete class (reference); useful to enable it only in testing:

```rust
#[cfg(test)]
use std::any::Any;

trait Trait {
  #[cfg(test)]
  // In some (non useful) cases, it's possible to get away without implementing this.
  fn as_any(&self) -> &dyn Any;
}

#[derive(Debug)]
struct Concr {}

impl Trait for Concr {
  #[cfg(test)]
  fn as_any(&self) -> &dyn Any {
    self
  }
}

#[test]
fn my_test() {
  let as_trait: Box<dyn Trait> = Box::new(Concr {});

  // Note the implicit Box dereferencing.
  //
  let _concr: &Concr = as_trait.as_any().downcast_ref::<Concr>().unwrap();
}
```

A reference to a trait **can't** be converted directly to a reference of a supertrait (upcasting):

```rust
// See https://stackoverflow.com/q/28632968.
//
let super_ref = &my_better_display as &fmt::Display;

// Basic workaround; for a more elaborate solution, see https://stackoverflow.com/a/28664881.
//
let super_ref = &(my_better_display as fmt::Display);
```

### Operator overloading

WATCH OUT! Think carefully about ownership - whether owned values (self/rhs) or references should be used.

```rust
struct Point(i32);
struct BigPoint(i32);

impl Add for Point {
  type Output = Self;

  // `rhs` stays for `Right Hand Side`
  //
  fn add(self, rhs: Self) -> Self::Output {
    Point(self.0 + rhs.0)
  }
}

impl Add<&BigPoint> for Point {
  type Output = Self;

  fn add(self, rhs: &BigPoint) -> Self::Output {
    Point(self.0 + 1000 * rhs.0)
  }
}

Point(10) + Point(20)
Point(10) + &BigPoint(1)
```

Some operators:

- `std::ops::Add/Sub/Mul/Div/Rem`: `self.add`, etc -> `Output` (`Rem`=%)
- `PartialEq`: `&self.eq`
- `std::ops::Neg`: `self.neg` -> `Output` (unary negation)

In order to overload on a trait, use the following syntax:

```rust
impl PartialEq for dyn Shape + '_ {
    fn eq(&self, rhs: &Self) -> bool {
        self.id() == rhs.id()
    }
}
```

### Method overloading (workaround)

Silly example. For basic data types, `Into<T>` has blanket implementations

```rust
struct Foo {
  value: Monster
}

impl Foo {
  fn set<T:Into<Monster>>(&mut self, value: T) {
    self.value += value.as_uint();
  }
}

impl Into<Monster> for Foo {
  fn into(self) -> Monster {
    self.monster
  }
}

impl Into<Monster> for Bar {
  fn into(self) -> Monster {
    self.monster
  }
}
```

### Associated functions/methods

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

// Multiple methods can be defined in the same `impl` block (and multiple blocks can be defined).
impl Rectangle {
    // When there is `self`, it's called "method".
    // `self` can also be borrowed mutably.
    // Methods taking ownership of `self` are rare and specific.
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    // Where there isn't `self`, it's called "associated function" (static method, in other langs).
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

Complex example ([source](https://dev.to/takaakifuruse/rust-lifetimes-a-high-wall-for-rust-newbies-3ap)). It seems that this example is not proper - `parse_context()` should borrow the context, not own it. However, this is interesting for the sake of understanding lifetimes.

```rust
struct Context<'c> {
  text: &'c str,
}

// The context outlives the parser.
struct Parser<'p, 'c> {
  context: &'p Context<'c>,
}

impl<'p, 'c> Parser<'p, 'c> {
  // This makes clear the lifetimes.
  fn new(context: &'p Context<'c>) -> Parser<'p, 'c> {
    Parser { context }
  }

  // The return value has the lifetime of the context.
  fn parse(&self) -> &'c str {
    &self.context.text
  }
}

// In order to make this possible, we need the two lifetimes. The `context` field of the `Parser` has
// the lifetime of the parameter, which is used in the return value of the function `parse()`, so that
// the `&str` is actually `&'c str`.
fn parse_context(context: Context) -> &str {
  Parser::new(&context).parse()
}

// The borrowed version doesn't need any special handling.

struct Context<'a> {
  text: &'a str,
}

struct Parser<'a> {
  context: &'a Context<'a>,
}

// Constructor omitted for simplicity.
impl<'a> Parser<'a> {
  fn parse(&self) -> &'a str {
    &self.context.text
  }
}

fn parse_context<'a>(context: &'a Context) -> &'a str {
  Parser { context }.parse()
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

// If the reference to the contained object is required, use:
//
let object_ref = Box::new(object).as_ref();
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

// `Rc::clone()` performs a shallow copy (as only the reference needs to be cloned); `clone()` is not
// appropriate, since it makes a deep copy.
//
let a = Rc::new(Parent(Rc::new(Parent(Rc::new(Nil)))));
{
  let _b = Parent(Rc::clone(&a));
  let _c = Parent(Rc::clone(&a));
  println!("Count: {}", Rc::strong_count(&a)); // 3
}
println!("Count: {}", Rc::strong_count(&a)); // 1
```

#### RefCell<T>/Mutex<T> and interior mutability

`RefCell<T>` allows mutating the contained value, even if the variable itself is immutable, therefore bypassing the compiler; generally speaking, it allows multiple owners while retaining mutability. The rules are enforced at runtime though, so this has an overhead.

`RefCell` is not `Sync`; in multithreaded context, `Mutex`/`RwLock` must be used instead.

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

#### Modifying Rc/Arc without inner mutable types (and conversion Box -> RC type)

If one wants to modify an RC type before sharing it, without inner mutable types, there is `get_mut()` (and unsafe/nightly `get_mut_unchecked()`). Example using the unchecked version and `Arc`:

```rust
pub fn new(mut children: Vec<Arc<Self>>) -> Arc<Self> {
  let mut parent = Arc::new(Self {
      parent: Weak::<Self>::new(),
      children: vec![],
  });

  for child in children.iter_mut() {
    let child_parent_ref = &mut unsafe { Arc::get_mut_unchecked(child) }.parent;
    *child_parent_ref = Arc::downgrade(&parent);
}

  let parent_mut = unsafe { Arc::get_mut_unchecked(&mut parent) };
  parent_mut.children = children;

  parent
}
```

Another option is to convert a `Box` to an RC type:

```rust
let mut boxed = Box::new(MyType { field: 64 });
boxed.field = 128;

let pizza = Into::<Arc<MyType>>::into(boxed);
let pizza: Arc<MyType> = boxed.into();
```

#### Weak<T> and reference cycles

Example with a tree data structure, with nodes pointing both to children and parents. The problem is that if we don't use weak references, there will be circular references (therefore leaks) because of parents pointing to children, and viceversa.

It's important to always think who is the owner. A parent ultimately owns the children - if the former is dropped, the children should be dropped too; therefore, the reference to the parent should be weak.

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
  value: i32,
  // RefCell is needed here, because Node will be wrapped in an Rc; without it, the contents can't
  // be mutated.
  //
  parent: RefCell<Weak<Node>>,
  children: RefCell<Vec<Rc<Node>>>,
}

// Can't instantiate `Weak` with a reference; must use `Rc::downgrade()`.
// `Weak::new()` is essentially a placeholder.
//
let leaf = Rc::new(Node {
  value: 3,
  parent: RefCell::new(Weak::new()),
  children: RefCell::new(vec![]),
});

let branch = Rc::new(Node {
  value: 5,
  parent: RefCell::new(Weak::new()),
  children: RefCell::new(vec![Rc::clone(&leaf)]),
});

// Common WATCH OUT! concepts:
//
// - don't forget to pass a reference when up/downgrading;
// - if a reference is borrowed, and must assign it, clone() it.

// When invoking `clone()`, it's important to use the for `(Weak|Rc)::clone(ref)`, otherwise, a deep
// clone is performed.
// For consistency with `Rc::downgrade()`, one can always use the type function `Weak::upgrade(ref)`,
// rather than `ref.upgrade()`.

// The `Rc`-enclosed type methods can be accessed via auto-dereferencing (`leaf.parent`).
//
// In order to change the content of a `RefCell`, use `borrow_mut()`.
//
// WATCH OUT! One may need to borrow and do the desired operation on the same statement, otherwise, BCK
// hilarity may ensue.
//
*leaf.parent.borrow_mut() = Rc::downgrade(&branch);

// In order to access a `Weak<T>` value, one must upgrade and `unwrap()` (if sure that there is a value).
//
println!("leaf parent = {:?}", Weak::upgrade(leaf.parent.borrow()).unwrap());

// When accessing an Option<Weak|Rc>, use the `if let`, or `clone().unwrap()`.
// Otherwise, an error is raised: `cannot move out of dereference of `RefMut<'_, ...>``
//
if let Some(ref mut child_wk) = opt_weak_ref { /* */ };
opt_weak_ref.clone().unwrap();

// Finally, Weak<dyn Trait> must receive a phony type on new() (on structs, one can use Self).
// See https://users.rust-lang.org/t/why-cant-weak-new-be-used-with-a-trait-object.
//
let parent: Mutex<Weak<dyn Shape>> = Mutex::new(Weak::<Plane>::new());

```

Counting functions:

```rust
Rc::strong_count(&branch);
Rc::weak_count(&branch);
```

##### `Rc<RefCell>` or `RefCell<Rc>`

This is not trivial. See:

1. https://stackoverflow.com/questions/57367092/what-is-the-difference-between-rcrefcellt-and-refcellrct
2. https://github.com/rust-lang/book/issues/1543#issuecomment-696995826

First case:

With `RefCell<Rc>`, `RefCell` can't be shared, and `Rc` can't be mutated.  
With `Rc<RefCell>`, `Rc` can be shared, and `RefCell` allows mutation.

Second case:

`children` (nodes) options:

```rust
// Vector is not shared, but it can be modified; the children are shared.
// Understanding: since the parent node is wrapped in an Rc, although the vector is not directly shared,
// it ends up being indirectly so.
//
RefCell<Vec<Rc<Node>>>

// Vector is shared; it can be modified, and the children, too.
// ??? -> When cloning, the list of nodes is shared, which is not what wanted.
//
Rc<RefCell<Vec<Node>>>

// Vector is shared, but can't be itself modified; instead, each child can be individually modified.
// This may not make much sense, since nodes can't be added/removed.
//
Rc<Vec<RefCell<Node>>>
```

A way to look at this is that what is between `Rc` and `RefCell` _can't_ be modified.

##### Real case of modeling a thread-safe tree with trait objects, with children addition

Example, with Tree implementing the Node trait. This is a **slow** implementation, since it will cause locking on the vector.

Read-only trees better do without locking at all (only Arc).

```rust
#[derive(SmartDefault)]
pub struct Tree {
  #[default(Mutex::new(Weak::<Self>::new()))]  // Note the (st00pid) necessary generic
  pub parent: Mutex<Weak<dyn Node>>,
  #[default(Mutex::new(vec![]))]
  pub children: Mutex<Vec<Arc<dyn Node>>>,
}

impl Tree {
  // This can't be an associate method, because we can't create a smart pointer to the parent (which
  // implies ownership), so we must pass a smart pointer with the parent, so that we can clone it.
  //
  pub fn add_child(parent: &Arc<dyn Node>, child: &Arc<dyn Node>) {
    parent.children().lock().unwrap().push(Arc::clone(child));
    let mut child_parent_ref = child.parent().lock().unwrap();
    *child_parent_ref = Arc::downgrade(parent);
  }
}
```

##### Real complex case of iterating a recursive structure with Rc/RefCell

See https://stackoverflow.com/questions/36597987/cyclic-reference-of-refcell-borrows-in-traversal.

```rust
struct LinkedList<T: Copy>(
  Option<(
    Rc<RefCell<Node<T>>>,
    RefCell<Node<T>>>
  )>
);

struct Node<T: Copy> {
  value: T,
  next: Option<Rc<RefCell<Node<T>>>>,
  previous: Option<Weak<RefCell<Node<T>>>>,
}

impl<T: Copy> LinkedList<T> {
  pub fn print_values(&self) -> Vec<T> {
    if let Some((ref front, _)) = self.0 {
      let mut current = Rc::clone(front);

      loop {
        println!("{}", current.borrow().value);

        // The are two crucial aspects.

        // 1. we can't borrow current outside of the `if let`, otherwise, the `current` assignment fails
        // because it's borrowed.
        //
        // let current_brw = current.borrow()
        // if let Some(_) = current_brw.next()

        let next = if let Some(ref next) = current.borrow().next {
          // 2. we need an `if let` construct, because if we assign current inside here, again, it's
          // borrowed. The solution here is to return an owned value, while releasing the borrow.

          Rc::clone(next)
        } else {
          break;
        };

        current = next;
      }
    }
  }
}
```

### Multithreading

Base usage:

```rust
use std::io::{self, Write};
use std::thread;
use std::time::Duration;

let message = "Hello";

// `move` is required to access variable in the context, with move semantics, because, the closure may
// outlive the referenced variable!
//
let handle = thread::spawn(move || {
  // Thread-safe way of printing.
  writeln!(&mut io::stdout().lock(), "{}", message).unwrap();
  thread::sleep(Duration::from_millis(1));
});

handle.join().unwrap();
```

If there are serious problems with lifetimes (eg. recursively passing references), use concurrent toolkits (see concurrency tools section).

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

WATCH OUT! Mutex is not reentrant, so multiple calls by the same thread (unless in the same function) will deadlock.

Note that an Arc can be safely modified if there is only one reference; see [the interior mutability section](#refcellt-and-interior-mutability).

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

Note that `Arc<Mutex<T>>` can be cloned and move around, but must be extremely careful, because it's very easy to deadlock.

The ownership of an instance in a mutex can be released, if there are no other threads holding the lock: `mutex_instance.into_inner().unwrap()`.

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

// Atomics can be globals!!!
//
static NEXT_ID: AtomicU32 = AtomicU32::new(1);

pub(crate) fn new_shape_id() -> u32 {
  NEXT_ID.fetch_add(1, Ordering::SeqCst)
}
```

Some APIs (return the previous value):

- `fetch_add(value, ordering)`:                add to the current value, and return the previous
- `compare_and_swap(current, value, ordering`: if the existing value equals `current`, set to `value`

**WATCH OUT**: Check out the [CPP reference](https://en.cppreference.com/w/cpp/atomic/memory_order) to understand memory orderings. The most conservative ordering, `Ordering::SeqCst` has still performance good enough to be used as default, according to Programming Rust 1st ed.

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

#### Busy waiting/spin loops (pause)

Intrinsic to use in busy waiting:

```rust
std::hint::spin_loop();
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

#### Function-like

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

// Define a struct.
//
macro_rules! matrix {
  ($name:ident, $order: literal) => {
    pub struct $name {
      pub values: [[u32; $order]; $order],
    }
  };
}

// Define variables (!)
//
macro_rules! vars {
  ($data:expr, $stride:expr, $var1:ident, $var2:ident, $var3:ident) => {
    let $var1 = $data[0];
    let $var2 = $data[1 * $stride];
  };
}

// Receive a statement; use a variable defined inside the macro
//
macro_rules! test_sort {
  ($collection:ident, $stat:stmt) => {
    #[cfg(test)]
    mod tests {
      use super::*;

      #[test]
      fn test_sort() {
        let mut $collection = &mut vec![3, 2, 1];
        $stat
        let expected_collection = &vec![1, 2, 3];
        assert_eq!($collection, expected_collection);
      }
    }
  };
}

test_sort!(collection, bubble_sort(collection));
```

##### Rules and details

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

##### Importing

Sample of importing macros across files:

```rust
// helpers.rs
//
#[macro_export]
macro_rules! test_sort {}

// main.rs
//
mod helpers;

// tests.rs
//
use crate::test_sort;
test_sort!(collection, bubble_sort(collection));
```

Macros are pulled in the main scope, so the `use` must not be followed by the macro enclosing module (`helpers`).

#### Custom derive macros

Procedural macros must be defined in a crate with the crate type of proc-macro.

```sh
cargo new macros --lib

# Convenience if in a workspace
perl -i -pe 's/members = \[\K/"macros", /' Cargo.toml

# Append straight below `[dependencies]`.
#
cat >> macros/Cargo.toml <<TOML
quote = "1.0"
syn = "1.0"

[lib]
proc-macro = true
TOML
```

`lib.rs` content; macro definitions *must* reside in the root crate:

```rust
// stdlib compiler API that allows manipulating Rust code.
extern crate proc_macro;

// syn converts a string into a data structure to work on; quote does the reverse.
use quote::quote;
use syn;

use proc_macro::TokenStream;

#[proc_macro_derive(MyMacro)]
pub fn my_macro_derive(input: TokenStream) -> TokenStream {
    // Replace unwrap() with better error handling on production code.
    let ast: syn::DeriveInput = syn::parse(input).unwrap();

    let name = &ast.ident;

    // stringify() converts an expression to string literal (&str), not String!
    let gen = quote! {
        // in order to import entities, rebind them
        use std::sync::Mutex as HelloMacroMutex;

        impl HelloMacro for #name {
            fn my_macro() {
                println!("{} my_macro() implementation via custom derive attribute!", stringify!(#name));
            }
        }
    };

    gen.into()
}
```

Now, from the client crate, just add the dependency (assumed to be in the same workspace):

```toml
macros = {path = "../macros"}
```

include in the project:

```rust
use macros::MyMacro;

// or (must be at the crate root)

#[macro_use]
extern crate macros; // name of the crate
```

then use!

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

### Assorted insanity

- Complex behavior of traits <> lifetimes (now obsolete; using `&Self` as argument is enough): https://stackoverflow.com/questions/54329200/mysterious-lifetime-issue-while-implementing-trait-for-dyn-object.

## Packaging

### Project structure

A package (=set of 1+ crates) can contain at most one library crate.

Crate "roots":

- `src/main.rs` -> binary crate (same name as package)
- `src/lib.rs` -> library crate (same name as package)

Multiple crates can be put in `src/bin`:

- `src/bin/mycrate.rs` -> binary crate (named `mycrate`)

Alternative configuration for multiple crates, via `Cargo.toml`:

```toml
# Array of tables -> there can be multiple.
#
[[bin]]
name = "daemon"
path = "src/daemon/bin/main.rs"
```

The relative (to the project root) source file path can be found via `file!()`.

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
- `super`: parent module (like current directory)
- `self`

Multiple files/directories structure:

```rust
// This loads either `module.rs` or `module/mod.rs`.
//
mod <module>;
```

Importing:

```rust
// The item imported must be public. Using doesn't make the item public!
// In order to import relatively, must (currently) use `self`.
//
// It's unidiomatic to import functions, while it's idiomatic to import enums, structs, etc.
//
use crate::front_of_house::hosting;

// Import sibling modules in sibling files (shape.rs). The first assumes no reexport; the second assumes
// a flattening reexport.
//
use super::shape::Shape;
use super::Shape;

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
SomeOptions { bar: true, ..Default::default() };
```

See [smart-default crate](#partialmore-flexible-defaults-smart-default) for partial/more flexible defaults.

### Copy, Clone, Drop and their relationships

See:

- https://stackoverflow.com/questions/51704063/why-does-rust-not-allow-the-copy-and-drop-traits-on-one-type
  - copy of `Copy` data is done via trivial `memcpy`; if drop was performed on a `Copy`+`Drop` copy, the original instance could include reference to invalid (not cleaned up) data
- https://www.reddit.com/r/rust/comments/8laxam/why_does_copy_require_clone
  - `Clone` is a supertrait of `Copy`

### Display

Simple implementation of Display:

```rust
impl std::fmt::Display for MyType {
  fn fmt(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
    write!(formatter, "value: {}", self.value)
  }
}
```

### From (/Into)

Implement when representing conversions from a type; the `Into` (`into()`) trait will be automatically handled.

```rust
struct Number {
  value: i32,
}

impl From<i32> for Number {
  fn from(item: i32) -> Self {
    Number { value: item }
  }
}

let num = Number::from(30);
let num: Number = int.into();
```

### Index[Mut]

Allow convenient map-like access to a struct.

```rust
struct Registers {
    SP: u16,
    PC: u16,
}

impl std::ops::Index<Register> for Registers {
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
