# Bevy

- [Bevy](#bevy)
  - [Chess game in Rust using Bevy](#chess-game-in-rust-using-bevy)
    - [Other resources](#other-resources)

## Chess game in Rust using Bevy

Source: https://caballerocoll.com/blog/bevy-chess-tutorial.

```rust
// The DefaultPlugins group includes the basic features to run a game; it makes a window popup because
// it includes WindowPlugin and WinitPlugin.
//
// Resources store data; WindowDescriptor stores the window properties.
//
App::build()
    .add_resource(WindowDescriptor {
        title: "Chess!".to_string(),
        width: 1600.,                             // WRITEME: can be i32 on Bevy 0.5
        height: 1600.,                            // ^^ same
        ..Default::default()
    })
    .add_plugins(DefaultPlugins)
    .run();
```

### Other resources

```rust
// Antialiasing
//
Msaa { samples: 4 }
```
