# Webassembly/Rust

- [Webassembly/Rust](#webassemblyrust)
  - [General](#general)
  - [Setup and Hello world](#setup-and-hello-world)
  - [Base concepts](#base-concepts)
    - [Rust side](#rust-side)
    - [JS side](#js-side)

## General

Notes from course [Snake Game With Rust, JavaScript, and WebAssembly](https://www.udemy.com/course/snake-game-with-rust-javascript-and-webassembly).

## Setup and Hello world

We call the package "wasm-snake-game".

wasm-pack part:

```sh
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Crate to install ready templates!
#
cargo install cargo-generate

cargo generate --git https://github.com/rustwasm/wasm-pack-template.git --name snake_rust_wasm-progress

# Build the project.
#
wasm-pack build
```

npm part:

```sh
npm init wasm-app www

cd www

# Add game dependency to the npm manifest.
#
perl -0777 -i -pe 's/\}\n\}$/},
  "dependencies": {
    "wasm-snake-game": "file:..\/pkg"
  }
}/' package.json

# Replace the js imported module.
#
perl -i -pe 's/from \K".*"/"wasm-snake-game"/' index.js

npm install
```

Hello world!!:

```sh
# Yeah!
#
npm start
xdg-open http://localhost:8080
```

## Base concepts

### Rust side

Import/export:

```rust
// Import a JS method
#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

// Export a Rust method
#[wasm_bindgen]
pub fn greet() {
  alert("Hello, snake-rust-wasm-progress!");
}

// Export a Rust struct
#[wasm_bindgen]
pub struct Vector {
  // Mark a method as constructor; adds `new Klazz()` form, otherwise, the error `Error: null pointer passed to rust`
  // is raised. See https://github.com/rustwasm/wasm-bindgen/issues/166.
  #[wasm_bindgen(constructor)]
  pub fn new() -> Self { /* ... */ }
}
```

### JS side

```js
// Import the exported entities.
//
import { Game, Vector } from "wasm-snake-game"

// Use them.
//
this.game = new Game(
  width, height,
  new Vector(snake_direction_x, snake_direction_y)
)
```
