# Bevy

- [Bevy](#bevy)
  - [Bevy book](#bevy-book)
    - [Setup](#setup)
    - [Hello world/Basic APIs](#hello-worldbasic-apis)

## Bevy book

### Setup

Increase compilation speed by using dynamic linking, but must disable for release!

```toml
bevy = { version = "0.5.0", features = ["dynamic"] }
```

### Hello world/Basic APIs

```rust
use bevy::prelude::*;

struct Person;

// ECS design: Don't add name as a field in Person - decouple it, so that it can be reused!
//
struct Name(String);

fn add_people(mut commands: Commands) {
  commands
    .spawn()
    .insert(Person)
    .insert(Name("Foo".to_string()));
  commands
    .spawn()
    .insert(Person)
    .insert(Name("Bar".to_string()));
  commands
    .spawn()
    .insert(Person)
    .insert(Name("Baz".to_string()));
}

// This is a simple system; systems run concurrently.
//
fn greet_people(query: Query<&Name, With<Person>>) {
  for name in query.iter() {
    if rand::random::<i32>() % 256 == 0 {
      println!("Hi {}!", name.0);
    }
  }
}

struct GreetTimer(Timer);

fn greet_people_timed(time: Res<Time>, mut timer: ResMut<GreetTimer>, query: Query<&Name, With<Person>>) {
  if timer.0.tick(time.delta()).just_finished() {
    for name in query.iter() {
      println!("Hello {}!", name.0);
    }
  }
}

pub struct HelloPlugin;

// Sample plugin; we bundle our systems here.
//
impl Plugin for HelloPlugin {
  fn build(&self, app: &mut AppBuilder) {
    let timer = GreetTimer(Timer::from_seconds(2.0, true));

    app.insert_resource(timer)
      .add_startup_system(add_people.system())
      .add_system(greet_people.system())
      .add_system(greet_people_timed.system());
  }
}

// Resources store data; WindowDescriptor stores the window properties.
//
// The DefaultPlugins group includes the basic features to run a game; it makes a window popup because
// it includes WindowPlugin and WinitPlugin. Since it also includes an event loop, the systems will
// run in an infinite loop.
//
App::build()
  .insert_resource(WindowDescriptor { title: "Hello W!".to_string(), width: 1280., height: 720., ..Default::default() })
  .add_plugins(DefaultPlugins)
  .add_plugin(HelloPlugin)
  .run();
```
