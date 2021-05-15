# Bevy

- [Bevy](#bevy)
  - [Setup](#setup)
  - [Hello world/Basic APIs](#hello-worldbasic-apis)
  - [General design/data structures](#general-designdata-structures)
    - [Components/bundles](#componentsbundles)
    - [Resources](#resources)
    - [Systems](#systems)
    - [Queries](#queries)
  - [Math APIs](#math-apis)

## Setup

Increase compilation speed by using dynamic linking, but must disable for release!

```toml
bevy = { version = "0.5.0", features = ["dynamic"] }
```

## Hello world/Basic APIs

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

## General design/data structures

### Components/bundles

Components:

```rust
// Use wrapper structs for simple data types ("newtype" pattern)
struct PlayerXp(u32);

// Use empty structs to mark entities (and query them)
struct Enemy;
```

Component Bundles are like templates for entities with common components:

```rust
// WATCH OUT! Don't forget the Bundle attribute, otherwise, if a bundle is accidentally used as component,
// no error is raised.
#[derive(Bundle)]
struct PlayerBundle {
  health: Health,
  _p: Player,

  // Bundles can be nested
  #[bundle]
  sprite: SpriteSheetBundle,
}

// Tuples of components are considered bundles
(ComponentA, ComponentB, ComponentC)
```

### Resources

Resources are used for globals, so be careful (use components as much as possible, as they can be shared). Any type (struct/enum) can be used

Definition:

```rust
// Use `Default` attribute for simple ones:
//
#[derive(Default)]
struct StartingLevel(usize);

// More complex types:
//
struct ComplexResource {
  // ... fields ...
}
impl FromWorld for ComplexResource {
  fn from_world(world: &mut World) -> Self {
    // This gives full access to anything in the ECS; e.g. mutating other resources:
    let mut x = world.get_resource_mut::<MyOtherResource>().unwrap();

    ComplexResource { /* ... */ }
  }
}
```

Instantiation:

```rust
// At app creation (extract):
//
App::build()
  // if it implements `Default` or `FromWorld`
  .init_resource::<ComplexResource>()
  // if not, or if you want to set a specific value
  .insert_resource(StartingLevel(3))

// From inside a system:
//
commands.insert_resource(GoalsReached { main_goal: false, bonus: false });
commands.remove_resource::<MyResource>();
```

### Systems

Systems are functions executed by the engine. Access (R/W) to the world from a given system is determined via function parameters:

```rust
// Tuples can be used to organize complex parameters:
//
fn complex_system(
  start: Res<StartingLevel>         // access resource
  (a, mut b): (Res<ResourceA>, ResMut<ResourceB>),
  mut c: Option<ResMut<ResourceC>>, // resource that may not exist
) {
  // ...
}
```

Systems are run by adding them when building the app:

```rust
App::build()
  .add_startup_system(init_menu.system())  // run it only once at launch
  .add_system(move_player.system())        // run it every frame update
}
```

### Queries

Used to access entity components. WATCH OUT! Bundles must be queried via component(s), not bundle type!

```rust
fn check_zero_health(
  // Find entities that have (`Health` & `Transform`) components, and are optionally a `Player`.
  // Entity is the entity id (not required).
  // With/out are (optional) filters: the entities must include an Ally, but not an Enemy.
  mut query: Query<
    (Entity, &Health, &mut Transform, Option<&Player>)>,
    (With<Ally>, Without<Enemy>)
) {
  // Iterate all matching entities
  for (eid, health, mut transform, player) in query.iter_mut() {
    if health.hp <= 0.0 { transform.translation = Vec3::ZERO; }
    if let Some(player) = player { /* the current entity is the player! */ }
  }
}

// Components for a specific entity.
//
if let Ok((health, mut transform)) = query.get_mut(entity) {
  // do something with the components
} else {
  // the entity does not have the components from the query
}

// Retrieve an entity that is guaranteed to be unique.
//
fn query_player(mut q: Query<(&Player, &mut Transform)>) {
  let (player, mut transform) = q.single_mut().expect("...");
}
```

## Math APIs

```rust
const_vec2!   // create a const Vec2; also for: Mat2/3/4, Quat, Vec3/3a/4
```
