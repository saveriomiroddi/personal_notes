# Webassembly/Rust

- [Webassembly/Rust](#webassemblyrust)
  - [General](#general)
    - [WASM Data types](#wasm-data-types)
  - [Setup + Hello world](#setup--hello-world)
  - [Base structure](#base-structure)
  - [Javascript side](#javascript-side)
  - [WASM-bindgen](#wasm-bindgen)

## General

Notes from book [Programming WebAssembly with Rust](https://pragprog.com/titles/khrust/programming-webassembly-with-rust).

Online IDE: https://webassembly.studio
Toolkit: https://github.com/WebAssembly/wabt

### WASM Data types

- `i32`/`i64`
- `f32`/`f64`

## Setup + Hello world

```sh
rustup target add wasm32-unknown-unknown

cargo new --lib rustwasmhello

cd rustwasmhello

# Specify a "C-style" library.
#
perl -i -pe 's/\[dependencies\]/[lib]
crate-type = ["cdylib"]

[dependencies]/' Cargo.toml

cat > src/lib.rs <<'RUST'
#[no_mangle]
pub extern "C" fn add_one(x: i32) -> i32 {
    x + 1
}
RUST

cargo build --release --target=wasm32-unknown-unknown
```

## Base structure

Add the dependencies `lazy_static` and `mut_static`.

Allow mutable global state; this is the easiest way to configure it:

```rust
extern crate mut_static;

#[macro_use]
extern crate lazy_static;

use game::GameEngine;
use mut_static::MutStatic;

lazy_static! {
  pub static ref GAME_ENGINE: MutStatic<GameEngine> = MutStatic::from(GameEngine::new());
}
```

Export functions to the host; since the interface with WASM is via basic datatypes (e.g. `i32`), communication must be performed through them:

```rust
#[no_mangle]
pub extern "C" fn get_current_turn() -> i32 {
  let engine = GAME_ENGINE.read().unwrap();
  GamePiece::new(engine.current_turn()).into()
}

impl Into<i32> for GamePiece {
  fn into(self) -> i32 {
    if self.color == PieceColor::Black { PIECEFLAG_BLACK } else { PIECEFLAG_WHITE }
  }
}
```

Declare functions imported from the host:

```rust
// Unsafe
extern "C" {
  fn notify_piecemoved(fromX: i32, fromY: i32, toX: i32, toY: i32);
  fn notify_piececrowned(x: i32, y: i32);
}
```

## Javascript side

```js
fetch('./rustycheckers.wasm').then(response =>
  response.arrayBuffer()
).then(bytes => WebAssembly.instantiate(bytes, {
  env: {
    // Imported functions; note that the namespace is `env`.
    //
    notify_piecemoved: (fX, fY, tX, tY) => {
      console.log("A piece moved from (" + fX + "," + fY + ") to (" + tX + "," + tY + ")");
    },
    notify_piececrowned: (x, y) => {
      console.log("A piece was crowned at (" + x + "," + y + ")");
    }
  },
}
)).then(results => {
  instance = results.instance;

  console.log("At start, current turn is " + instance.exports.get_current_turn());
  let piece = instance.exports.get_piece(0, 7);
  let res = instance.exports.move_piece(0, 5, 1, 4);
  let bad = instance.exports.move_piece(1, 4, 2, 3);
}).catch(console.error);
```

## WASM-bindgen

Install the crate `wasm-bindgen-cli`, then add the `wasm-bindgen` dependency.

This crate allows easy bindings:

```rust
extern crate wasm_bindgen;
use wasm_bindgen::prelude::*;

// Import 'window.alert'
#[wasm_bindgen]
extern "C" {
  fn alert(s: &str);
}

// Export a 'hello' function
#[wasm_bindgen]
pub fn hello(name: &str) {
  alert(&format!("Hello, {}!", name));
}
```

Produce the module and JS wrapper file:

```sh
# Creates `bindgenhello_bg.wasm` and `bindgenhello.js`
wasm-bindgen target/wasm32-unknown-unknown/debug/bindgenhello.wasm --out-dir .
```
