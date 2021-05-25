# Rust libraries

- [Rust libraries](#rust-libraries)
  - [Standard library](#standard-library)
    - [Files/streams](#filesstreams)
    - [Paths handling/Directories](#paths-handlingdirectories)
    - [Testing](#testing)
      - [Integration tests](#integration-tests)
    - [String/char-related (conversions)](#stringchar-related-conversions)
    - [Collections](#collections)
      - [BTreeSet: sorted set](#btreeset-sorted-set)
      - [VecDeque: double-ended queue](#vecdeque-double-ended-queue)
    - [TCP client/server](#tcp-clientserver)
    - [Commandline arguments (basic)](#commandline-arguments-basic)
    - [O/S, Processes](#os-processes)
    - [Blackbox (nightly)](#blackbox-nightly)
  - [Crates](#crates)
    - [Partial/more flexible defaults (`smart-default`)](#partialmore-flexible-defaults-smart-default)
    - [Random (with and without `rand`)](#random-with-and-without-rand)
    - [Regular expressions (`regex`)](#regular-expressions-regex)
    - [Date/times (standard)](#datetimes-standard)
    - [Date/times (`chrono`)](#datetimes-chrono)
    - [Commandline parsing (`clap`)](#commandline-parsing-clap)
    - [Map literals (`maplit`)](#map-literals-maplit)
    - [Channels: Single Producer Multiple Consumers (`bus`)](#channels-single-producer-multiple-consumers-bus)
    - [Unit testing](#unit-testing)
      - [RSpec-style testing (`demonstrate`)](#rspec-style-testing-demonstrate)
      - [Cucumber](#cucumber)
      - [Utilities](#utilities)
    - [Static/global variables (`lazy_static`, `once_cell`, `thread_local!`)](#staticglobal-variables-lazy_static-once_cell-thread_local)
    - [Concurrency (multithreading) tools (`rayon`/`crossbeam`)](#concurrency-multithreading-tools-rayoncrossbeam)
    - [Enum utils, e.g. iterate (`strum`)](#enum-utils-eg-iterate-strum)
    - [Convenience macros for operator overloading (`auto_ops`)](#convenience-macros-for-operator-overloading-auto_ops)
    - [Indented Heredoc-like strings (`indoc`)](#indented-heredoc-like-strings-indoc)
    - [User directories (`directories`)](#user-directories-directories)

## Standard library

### Files/streams

```rust
std::fs::read_to_string(filename) -> Result<String, Error>;    // content must be valid UTF-8; filename can be relative.
std::fs::read(game_rom_filename) -> Result<Vec<u8>, Error>;    // read binary content
writer.write_all(data: AsRef<[u8]>) -> Result<()>              // write to a writer (e.g. File)

// Buffered read/write require the import below
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
let lines = reader.lines().collect::<Result<Vec<_>, _>>()?;

// Open file for writing; if existing, it's truncated.
//
let mut file = File::create("log.txt")

// Buffered write. Think about write() vs. write_all()
// Flushes when the buffer is full, not at the end of each line.
//
let mut stream = BufWriter::new(TcpStream::connect("127.0.0.1:34254").unwrap());
stream.write(&[666]).unwrap();

// Per-line buffered write; convenient if each line must be immediately available.
let mut writer = LineWriter::new(file);
writer.write_all(b"I like pizza!\n")?;
```

Abstract operation traits:

- `std::io::Read`
- `std::io::Write`: `write_all(buf: &[u8])`, `write(buf: &[u8])`
  - prefer `write_all()` to `write()`, since the latter doesn't guarantee that the whole buffer is written!

`Vec` can be trivially used as `StringIO` equivalent:

```rust
BufReader::new(&str.as_bytes());
BufReader::new(vec.as_slice()); // don't forget that Read requires slices!
BufWriter::new(vec);
```

for more complex operations (ie. involving seek), can use [io::Cursor](https://doc.rust-lang.org/std/io/struct.Cursor.html).

### Paths handling/Directories

(for user paths/directories, see the crates section)

For paths handling, use `std::path::Path`, with several conveniences:

```rust
// PathBuf is the owned type.
//
let p: PathBuf = PathBuf::from("/path/to/file");
let p: PathBuf = Path::new(ASSETS_PATH).join("triangles.obj");
let p: PathBuf = Path::new('/path').to_owned();

// Methods common to Path and PathBuf.
//
let p: Option<&OsStr> = path.file_name(); // Ruby basename(); for the poor man's version, use String#split
let p: Option<&OsStr> = path.file_stem(); // Filename without extension/path. Must do the (very)

// Conversions (methods available to both Path/PathBuf):
//
// - to borrowed types is easy;
// - to owned (String) is easy (but lossy) via to_string_lossy() (which returns Cow<str>).
//
let p: &OsStr         = path.as_os_str();
let p: Option<&str>   = path.to_str();
let p: String         = path.to_string_lossy().into_owned();

// The rigorous conversions are quite ugly.
// OsString doesn't implement fmt::Display, however, it implements fmt::Debug.
//
pathbuf
  .into_os_string() // OsString
  .into_string()    // Result<String, OsString>
  .unwrap();

pathbuf
  .file_name()      // Option<&OsStr>
  .unwrap()         // &OsStr
  .to_owned()       // OsString
  .into_string()
  .unwrap();

// From borrowed version, as_os_str().to_str() is an alternative, although, it may be conceptually
// simpler just to go through to_owned().
//
path
  .as_os_str()      // &OsStr
  .to_str()         // Option<&str>
  .unwrap()         // &str
  .into_string();   // String
```

Directories:

```rust
path.exists()                    // test if file/dir exists
std::fs::create_dir(&path)?;     // create a directory (mkdir)
std::fs::create_dir_all(&path)?; // create a directory (mkdir -p)
```

### Testing

WATCH OUT! UTs are run in parallel by default!! In order to run specific tests serially, must use the [serial_test crate](https://crates.io/crates/serial_test).

```rust
fn my_method() -> u32 {
  1
}

#[cfg(test)]
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
cargo test -- --test=test-threads=1  # run serially
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

### String/char-related (conversions)

Conversions:

```rust
integer.to_string();                      // integer to string
String::from_utf8(bytes).unwrap();        // string from (valid) utf-8 bytes

// parse string to numeric type; with any numeric implementing `FromString`
// f64 will parse integer strings (e.g. `1`)
//
let guess: u32 = string.parse().unwrap();
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

### Collections

#### BTreeSet: sorted set

```rust
let set = BTreeSet::new()

set.insert(10);
set.insert(4);
set.insert(1981);

set.contains(88); // false
set.remove(123);  // false
set.clear();

set.first();      // 4
set.last();       // 1981

set.is_subset(&BTreeSet);
set.is_superset(&BTreeSet);
set.is_disjoint(&BTreeSet);  // disjoint: no elements in common
set.intersection(&BTreeSet).clone().collect();
set.union(&BTreeSet).clone().collect();
```

For sorting floats, see [sorting section](#sorting-floats).

#### VecDeque: double-ended queue

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

Don't forget that the first is the binary filename.

```rust
std::env::args();                                       // only valid Unicode; can collect to Vec<String>
std::env::args_os();                                    // returns `OsString`s, which are not restricted to Unicode
let exe: io::Result<PathBuf> = std::env::current_exe(); // current binary/executable path
let exe: io::Result<PathBuf> = std::env::current_dir(); // working directory
```

### O/S, Processes

```rust
std::env::consts::OS;               // values: https://doc.rust-lang.org/std/env/consts/constant.OS.html
std::process::exit(exit_status);    // terminate program (exit)
```

### Blackbox (nightly)

Be pessimistic about the side effects of this function. Can't make any absolute guarantee.

```rust
#![feature(test)]

use test::bench::black_box;

pub fn black_box<T>(dummy: T) -> T
```

## Crates

### Partial/more flexible defaults (`smart-default`)

```rust
#[derive(SmartDefault)]
pub struct Sphere {
  #[default(_code = "Self::new_id()")] // the method can be private
  pub id: u32,
  #[default(Matrix::identity(4))]
  pub transformation: Matrix,
  #[default(Material::default())]
  pub material: Material,

  // All the fields need to have a default; the non-meaningful defaults can be overridden on an instantiation
  // method, e.g. new().
}

let sphere1 = Sphere::default();

let mut sphere2 = Sphere {
  material: Material { /* .. */ },
  ..Sphere::default()
};
```

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

With standard crate `rand` (see other crates at https://sr.ht/~icefox/oorandom):

```rust
// Shortcut API; invokes `thread_rng().gen()`.
//
let rand_byte: u8 = rand::random();

// Ends: [low, high); requires `rand::Rng`.
//
let randval = match rand::thread_rng().gen_range(0..2) {
    0 => "0",
    1 => "1",
    _ => unreachable!(),
};

// Fetching a random element from an array
//
use rand::seq::SliceRandom;
let entry: Option<&MyType> = vec.choose(&mut rand::thread_rng());

// Use thread_rng
// `thread_rng()`: seeded by the O/S; local to the current thread.
//
use rand::prelude::*;
let mut rng = rand::thread_rng();
let randval: f64 = rng.gen();

let mut data = [0u8; 32];
rand::thread_rng().fill_bytes(&mut data);
```

Use a deterministic generator, for testing purposes:

```rust
// Bools are converted from (little endian) i32 (4 bytes); the highest bit determines the value.

use rand::{rngs::adapter::ReadRng, Rng, RngCore};

let data = [true, false, true, true, false, false, true]
    .iter()
    .flat_map(|f| vec![0, 0, 0, (*f as u8) << 7])
    .collect::<Vec<_>>();

let mut rng: Box<dyn RngCore> = Box::new(ReadRng::new(data.as_slice()));
let rand_bool = rng.gen::<bool>();

// Sample type with user-definable RNG.
//
struct MyRng<'a> {
  rng: Box<dyn RngCore + 'a>,
}
impl<'a> MyRng<'a> {
  pub fn new(rng: Option<Box<dyn RngCore + 'a>>) -> Self {
    let rng = rng.unwrap_or_else(|| Box::new(thread_rng()));
    MyRng { rng }
  }
}
```

### Regular expressions (`regex`)

```rust
let re = Regex::new(r"(\d{4})(\d{2})").unwrap();
let text = "201203, 201301, 201407";

re.is_match(text); // true

// WATCH OUT!: the capture 0 is the whole string (as standard).

for cap in re.captures_iter(text) {
    println!("Whole: '{}', $1: '{}', $2: '{}'", &cap[0], &cap[1], &cap[2]);
}

// COOL!!!! Convert to Vec<_>, and pattern match.
//
let value = captures
  .iter()
  .skip(1)                       // skip global capture
  .map(|c| c.unwrap().as_str())  // use `c.map(|m| m.as_str())` if the captures are NOT guaranteed
  .collect::<Vec<_>>();

// If the captures are not guaranteed, must handle the `Option<_>`
//
if let [v1, n1, v2, n2, v3, n3] = values.as_slice() {
  FaceWithNormal((*v1, *n1), (*v2, *n2), (*v3, *n3))
} else { unreachable!() }
```

Match multiple regexes:

```rust
lazy_static::lazy_static! {
  static ref VERTEX_REGEX: Regex = Regex::new(r"^v (-?1(?:\.\d+)) (-?1(?:\.\d+)) (-?1(?:\.\d+))$").unwrap();
  static ref FACE_REGEX: Regex = Regex::new(r"^f (\d+) (\d+) (\d+)$").unwrap();
}

if let Some(captures) = VERTEX_REGEX.captures(&line) {
  let x: f64 = captures[1].parse().unwrap();
  // ...
} else if let Some(captures) = FACE_REGEX.captures(&line) {
  // ...
} else {
  // ...
}
```

### Date/times (standard)

The Instant counter doesn't stop when a sleep is issued.

```rust
// Compute time elapsed.
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

WATCH OUT! Check accurately how the chrono crate is more convenient than the stdlib version.

WATCH OUT! `std::time::Duration` doesn't support negative values.

```rust
let start:DateTime<Utc> = Utc::now();                   // Current time

DateTime::parse_from_str(string, "%d.%m.%Y %H:%M %P %z")?;   // Parse a datetime, with timezone
NaiveTime::parse_from_str(string, "%H:%M:%S")?;              // Parse a time
NaiveDateTime::parse_from_str(string, "%Y-%m-%d %H:%M:%S")?; // Parse a datetime, without timezone

start + Duration::days(3);                              // Arithmetic
start.checked_add(Duration::days(3));                   // Safe arithmetic
end.checked_sub(start);                                 // Returns Option<Duration> (none if negative)
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

### Unit testing

If files needs to be opened, the current path is the root of the current project (not workspace root!).

Standard testing:

```rust
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
```

#### RSpec-style testing (`demonstrate`)

Repository [here](https://github.com/austinsheep/demonstrate).

```rust
use demonstrate::demonstrate;

// If a single instance needs to be shared between UTs ("before all"), then trickery is needed.
// The Mutex is required for mutability.
//
// A simpler approach to this is to use an empty mutex (`Mutex<()>`) and initialize on each UT.
//
use std::sync::Mutex;
unsafe impl Send for MyInterface {}
lazy_static! {
  static ref INTERFACE: Mutex<MyInterface> = Mutex::new(MyInterface::init("test"));
}

demonstrate! {
  // Not needed in this case; here for reference.
  use super::*;

  describe "test module 2" {
    before { let context = 5; }
    subject { context + 5 }

    it "used a shared object" {
      INTERFACE().lock().unwrap().read_pixel(0, 0)
    }

    it "test subject" {
      assert_eq!(subject, 10);
    }

    #[should_panic]
    it "should fail" {
        None::<()>.unwrap();
    }

    context "context is 5" {
      it "should equal 5" {
        assert_eq!(context, 5);
      }
    }
  }
}
```

There is a [Cucumber crate](https://github.com/bbqsrc/cucumber-rust). As of Nov/2020, the README tutorial [is broken](https://github.com/bbqsrc/cucumber-rust/issues/90), so see [another tutorial](https://www.florianreinhard.de/2020-10-05/cucumber-07-in-rust-beginners-tutorial).

#### Cucumber

There are two options - the stable version, and the master version, which has macros that make the test structure arguably more readable; this section uses the latter.

Update the configuration:

```sh
cat >> Cargo.toml <<TOML
async-trait = "0.1.42" # This is currently required to properly initialize the world in cucumber-rust
futures = "0.3.8" # You can use a different executor if you wish

[[test]]
name = "cucumber"
harness = false # Allows Cucumber to print output instead of libtest

[dev-dependencies]
cucumber_rust = { git = "https://github.com/bbqsrc/cucumber-rust", branch = "main", features = ["macros"] }
TOML
```

Add the feature(s):

```sh
mkdir features

cat > features/examples.feature <<CUCUMBER
Feature: Example feature

  Scenario: An example scenario
    Given I am trying out Cucumber
    When I consider what I am doing
    Then I am interested in ATDD
    And we can implement rules with regex
CUCUMBER
```

and the test(s):

```sh
mkdir tests

cat > tests/cucumber.rs <<'RUST'
use std::{cell::RefCell, convert::Infallible};

use async_trait::async_trait;
use cucumber_rust::{given, then, when, World, WorldInit};

#[derive(WorldInit)]
pub struct MyWorld {
    // You can use this struct for mutable context in scenarios.
    foo: String,
    bar: usize,
    some_value: RefCell<u8>,
}

impl MyWorld {
    async fn test_async_fn(&mut self) {
        *self.some_value.borrow_mut() = 123u8;
        self.bar = 123;
    }
}

#[async_trait(?Send)]
impl World for MyWorld {
    type Error = Infallible;

    async fn new() -> Result<Self, Infallible> {
        Ok(Self {
            foo: "wat".into(),
            bar: 0,
            some_value: RefCell::new(0),
        })
    }
}

#[given("a thing")]
async fn a_thing(world: &mut MyWorld) {
    world.foo = "elho".into();
    world.test_async_fn().await;
}

#[when(regex = "something goes (.*)")]
async fn something_goes(_: &mut MyWorld, _wrong: String) {}

#[given("I am trying out Cucumber")]
fn i_am_trying_out(world: &mut MyWorld) {
    world.foo = "Some string".to_string();
}

#[when("I consider what I am doing")]
fn i_consider(world: &mut MyWorld) {
    let new_string = format!("{}.", &world.foo);
    world.foo = new_string;
}

#[then("I am interested in ATDD")]
fn i_am_interested(world: &mut MyWorld) {
    assert_eq!(world.foo, "Some string.");
}

#[then(regex = r"^we can (.*) rules with regex$")]
fn we_can_regex(_: &mut MyWorld, action: String) {
    // `action` can be anything implementing `FromStr`.
    assert_eq!(action, "implement");
}

fn main() {
    let runner = MyWorld::init(&["./features"]);
    futures::executor::block_on(runner.run_and_exit());
}
RUST
```

Fire!:

```sh
cargo test --test cucumber
```

#### Utilities

```rust
// `serial_test` allows marked tests to be run serially; works with `demonstrate`!
//
#[serial]
it "mytest" { /* ... */ }

#[serial]
it "mytest2" { /* ... */ }

// assert_float_eq: nice, but doesn't support messages
//
assert_float_absolute_eq!(3.0, 3.0);      # default epsilon = 1e-6
assert_float_absolute_eq!(3.0, 3.9, 1.0);

// The [Spectral](https://github.com/cfrancia/spectral) create has fluent assertions
//
asserting(&"test condition").that(&1).is_equal_to(&2);
```

### Static/global variables (`lazy_static`, `once_cell`, `thread_local!`)

Static variables can be defined inside a function (c-style) or at the root level; they need a constant expression/etc. They are unsafe when mutable.

The difference from const is that they're associated only one memory location.

```rust
static HELLO_WORLD: u32 = 1000;
```

Global variables; also, allows initializing static variables with any function.

```rust
lazy_static::lazy_static! {
  static ref HASHMAP: HashMap<u32, &'static str> = {
    let mut m = HashMap::new();
    m.insert(0, "foo");
    m
  };
}
```

In order to mutably access, a `Mutex` (or `RWLock`) needs to be wrapped around.

Same, with another crate, `once_cell` (also in unstable, as `SyncLazy`):

```rust
static ARRAY: Lazy<Mutex<Vec<u8>>> = Lazy::new(|| Mutex::new(vec![]));
```

Thread-local variables:

```rust
// In stdlib.
//
thread_local!(static TL_VAR: RefCell<u32> = RefCell::new(123));

TL_VAR.with(|tl_var_cell| {
  let mut tl_var_ref = tl_var_cell.borrow_mut();
  *tl_var_ref = 32;
});

TL_VAR.with(|tl_var_cell| {
  let tl_value = tl_var_cell.borrow();
  println!("TLV: {}", tl_value); // 32
});
```

### Concurrency (multithreading) tools (`rayon`/`crossbeam`)

Easy parallel iteration!!!:

```rust
use rayon::prelude::*;

// Use this in order to limit the used threads (or use the `RAYON_NUM_THREADS` env var); by default,
// all the hw threads are used.
//
rayon::ThreadPoolBuilder::new()
    .num_threads(8)
    .build_global()
    .unwrap();

array.par_iter()
     .map(|&i| i * i)
     .sum()

range.into_par_iter().for_each(|v| { /* parallel operation */ });
```

Both Rayon and [Crossbeam](https://github.com/crossbeam-rs/crossbeam) provide tools for concurrent programming:

```rust
// Threads pooling. Queue size is chosen automatically based on cores.
//
rayon::join(|| quicksort(a), || quicksort(&mut b[1..]))
```

### Enum utils, e.g. iterate (`strum`)

```rust
// include both `strum` and `strum_macros`.

use strum_macros::EnumIter;

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Flag {
  z,
  n,
  h,
  c,
}

use strum::IntoEnumIterator;

for flag in Flag::iter() { /* ... */ }
```

### Convenience macros for operator overloading (`auto_ops`)

See [repository](https://github.com/carbotaniuman).

### Indented Heredoc-like strings (`indoc`)

```rust
use indoc::indoc;

// Ruby squiggly heredoc-alike syntax.
//
let expected_string = indoc! {"
    P3
    5 3
    255
    0 0 0 0 0 0 0 0 0 0 0 0 0 0 255 0 0 0 0 0 0 0 128 0 0 0 0 0 0 0 255 0
    0 0 0 0 0 0 0 0 0 0 0 0 0
"};
```

### User directories (`directories`)

Aside the temporary directory, there's not builtin way in Rust to find the home (and related) user directories.

Generic create search: https://crates.io/search?q=home

```rust
// stdlib temp dir
//
let tmpdir: PathBuf = std::env::temp_dir();

// `directories` crate
//
UserDirs::new().home_dir();
UserDirs::new().desktop_dir();
```
