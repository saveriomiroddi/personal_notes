# Webassembly/Rust

- [Webassembly/Rust](#webassemblyrust)
  - [General](#general)
  - [Setup and Hello world](#setup-and-hello-world)
  - [Base concepts](#base-concepts)
    - [WASM Data types/structures](#wasm-data-typesstructures)
    - [Rust side](#rust-side)
    - [JS side](#js-side)
  - [Deploy (with preparation)](#deploy-with-preparation)

## General

The notes are:

1. mainly from course [Snake Game With Rust, JavaScript, and WebAssembly](https://www.udemy.com/course/snake-game-with-rust-javascript-and-webassembly);
2. in little part, from [Programming WebAssembly with Rust](https://pragprog.com/titles/khrust/programming-webassembly-with-rust).

These notes use the approach of the first resource - `wasm-pack` and templates as base, in order to simplify the development; the second resource sets the project elements manually.

## Setup and Hello world

The package is named, as example, `wasm-snake-game`.

wasm-pack part:

```sh
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Crate to install ready template.
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

### WASM Data types/structures

- `i32`/`i64`
- `f32`/`f64`

It's possible to export structs and enums; add `js-sys` dependnecy for bindings to JS standard objects:

```sh
perl -i -pe 's/^\[dependencies\]\K/
js-sys = "0.3.32"
/' Cargo.toml
```

### Rust side

Import/export:

```rust
// Import a JS method
//
#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

// Export a Rust method
//
#[wasm_bindgen]
pub fn greet() {
  alert("Hello, snake-rust-wasm-progress!");
}

// Export a Rust struct
//
#[wasm_bindgen]
pub struct Vector {
  // Mark a method as constructor; adds `new Klazz()` construct; otherwise, the error `Error: null pointer passed to rust`
  // is raised. See https://github.com/rustwasm/wasm-bindgen/issues/166.
  // Other struct methods don't need the attribute.
  //
  #[wasm_bindgen(constructor)]
  pub fn new() -> Self { /* ... */ }
}
```

Bindings:

```rust
// In order to build a Rust array of Point that can be used on the JS side, we need to convert from
// `Vec<Point>` to `Array<JsValue>`.

use js_sys::Array;

// This is a convenience to convert &Point to JsValue.
//
impl From<&Point> for JsValue {
  fn from(point: &Point) -> Self {
    JsValue::from(point.clone())
  }
}

#[wasm_bindgen]
impl Game {
  pub fn get_snake(&self) -> Array {
    self.snake.iter().map(JsValue::from).collect()
  }
}
```

### JS side

Example(s) of using the Rust important entities:

```js
// Import the exported entities.
//
import { Game, Point } from "wasm-snake-game"

// Use them.
//
this.head = new Point(snake_direction_x, snake_direction_y)

snake = this.game.get_snake()
snake
  .map(this.projectPosition)
  .forEach(({ x, y }) => this.context.lineTo(x, y))
```

## Deploy (with preparation)

Preparation:

```sh
cd www

# Install the tool for easily deploy to GitHub pages.
#
npm install --save-dev gh-pages

# Add webpack task (to the "scripts" node).
#
perl -0777 -i -pe 's/"scripts": \{$\K/
    "deploy": "gh-pages -d dist",
/' package.json

# Set the repository.
#
perl -i -pe 's/"url": "git\+\K.*/https:\/\/github.com\/saveriomiroddi\/snake_rust_wasm-progress.git"/' package.json

# "url": "git+https://github.com/rustwasm/create-wasm-app.git"
```

Build and deploy!:

```sh
wasm-pack build --release
npm run deploy
```

Don't forget to enable the GitHub pages for the repository.
