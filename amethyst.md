# Amethyst

- [Amethyst](#amethyst)
  - [Build book](#build-book)
  - [Concepts](#concepts)
    - [States](#states)
    - [Game data](#game-data)
    - [State/date example](#statedate-example)
    - [ECS general info](#ecs-general-info)
    - [Components implementation](#components-implementation)
    - [Storages](#storages)
    - [Resources](#resources)
    - [World/management](#worldmanagement)
    - [Systems](#systems)

## Build book

```sh
# May be required on master.
perl -i -pe 's/^follow-web-links = \K/false/' book/book.toml

mdbook build book

ln -s "$PWD"/book/book/index.html ~/Desktop/amethyst_book.html
```

## Concepts

### States

States can be handled in two ways:

- as stack: states are pushed; if one is popped, the current state becomed the current top
- the current state can be switched with another; the switch is called `Transition`, and is triggered by an `Event`

State callbacks:

- `on_start`: State added to the stack
- `on_stop`: State removed from the stack
- `on_pause`: Another state pushed on top; current state is paused
- `on_resume`: Another state popped; the new state becomes active and resumes
- `handle_event`: generic events handling, e.g. window closing or keypress
- `fixed_update`: called on the active state at a fixed time interval (1/60th second by default).
  - `shadow_fixed_update`: called on all the states in the stack (incl. the active state) at a fixed time interval (1/60th second by default); unlike `fixed_update`, this does not return a `Trans`.
- `update`: called on the active state as often as possible by the engine.
  - `shadow_update`: called on all states in the stack (incl. the active state) as often as possible by the engine; unlike `update`, this does not return a `Trans`.

If you aren't using `SimpleState` or `EmptyState`, you must implement the update method to call `data.data.update(&mut data.world)`.

### Game data

`State`s can include associated data; if tighly coupled, it can go in the `State`'s struct.

`State`s also have internal data, which is any type `T`. In most cases, the two following are the most used:

- `()`: no data associated; usually used for tests and not for actual games;
- `GameData` is the standard; it contains a `Dispatcher`.

When calling your `State`'s methods, the engine will pass a `StateData` struct which contains both the `World` and the `GameData` type.

### State/date example

Trivial state:

```rust
struct GameplayState {
  // State-local data
  player_count: u8,
}

impl SimpleState for GameplayState {
  fn on_start(&mut self, _data: StateData<'_, GameData<'_, '_>>) {
    println!("Number of players: {}", self.player_count);
  }
}
```

State transition:

```rust
struct GameplayState;
struct PausedState;

// Instead of writing `State<(), StateEvent>`, we can instead use `EmptyState`.
impl EmptyState for GameplayState {
  fn handle_event(&mut self, _data: StateData<()>, event: StateEvent) -> EmptyTrans {
    if let StateEvent::Window(event) = &event {
      if is_key_down(&event, VirtualKeyCode::Escape) {
        return Trans::Push(Box::new(PausedState));
      }
    }

    Trans::None
  }
}

impl EmptyState for PausedState {
  fn handle_event(&mut self, _data: StateData<()>, event: StateEvent) -> EmptyTrans {
    if let StateEvent::Window(event) = &event {
      if is_key_down(&event, VirtualKeyCode::Escape) {
        // Go back to the `GameplayState`.
        return Trans::Pop;
      }
    }

    Trans::None
  }
}
```

Select the events that a state receives:

```rust
#[derive(Debug, EventReader, Clone)]
#[reader(MyEventReader)]
pub enum MyEvent {
  Window(Event),
  UI(UiEvent),
  App(AppEvent),
}

struct GameplayState;

impl State<(), MyEvent> for GameplayState {
  fn handle_event(&mut self, _data: StateData<()>, event: MyEvent) -> Trans<(), MyEvent> {
    match event {
      MyEvent::Window(_) => { /* Events related to the window and inputs */ },
      MyEvent::UI(_) => { /* UI event. Button presses, mouse hover, etc. */ },
      MyEvent::App(ev) => println!("Got an app event: {:?}", ev),
    };

    Trans::None
  }
}

// In order for the above to work, the app must be build this way:
//
CoreApplication::<_, MyEvent, MyEventReader>::build();
CoreApplication::<_, MyEvent, MyEventReader>::new();
```

### ECS general info

Entities are trivial: `struct Entity(u32, Generation);`

Say, we have:

- `Person`: `(x, y)`, `name`
- `Bottle`: `(x, y)`, `shape`

The properties (of all the objects) could be stored in the three storages:

- `PositionComponent`: `(x,y)`
- `PersonComponent`: `name`
- `BottleComponent`: `shape`

This is very similar to an RDBMS (joins are indeed performed).

### Components implementation

```rust
// Shape of an `Entity`
enum Shape {
  Sphere { radius: f32 },
  Cuboid { height: f32, width: f32, depth: f32 },
}

// Transform of an `Entity`
pub struct Transform {
  isometry: Isometry3<f32>,
  scale: Vector3<f32>,
}

// Storage is lazily initialized - on registration or usage.

impl Component for Shape {
  type Storage = DenseVecStorage<Self>;
}

impl Component for Transform {
  type Storage = FlaggedStorage<Self, DenseVecStorage<Self>>;
}
```

### Storages

- `DenseVecStorage`: Contiguous (no empty space) vector, good for big components or as generic default;
- `VecStorage`: Sparse vector, good for small (<= 2⁴ bytes) components or when carried by most (> 30%) entities;
- `FlaggedStorage`: Tracks components changes, good for caches;
- `NullStorage`: Used for tagging (tag = empty struct).


### Resources

Resources are types that are not specific to an entity (eg. score); they can be used by shared between systems.

Resource example: sprites loader.

```rust
// Amethyst assets loader.
//
let loader: Fetch<Loader> = world.read_resource::<Loader>();
```

### World/management

```rust
let mut world = World::empty();
```

Resources handling:

```rust
struct MyResource {
  pub game_score: i32,
}

// Insert
//
let resource = MyResource { game_score: 0 };
world.insert(resource);

// Fetch, fetch/insert, try fetch
//
let resource = world.read_resource::<MyResource>();
let resource = world.entry::<MyResource>().or_insert_with(|| resource);
let resource: Option<Fetch<_>> = world.try_fetch::<MyResource>();

// Mutable: Fetch, try fetch
//
let mut resource = world.write_resource::<MyResource>();
let mut resource: Option<FetchMut<_>>  = world.try_fetch_mut::<MyResource>();

// Deletions can't be done directly. Instead, add an Option<MyResource>, then set it to None to delete.
```

Entities handling:

```rust
// Create
//
let entity = world
  .create_entity()
  .with(MyComponent)
  .build();

// Get all
//
let entities: EntitiesRes = world.entities();

// Delete one/many/all entities and their components
// WATCH OUT!: Entities are lazily deleted; deletion only happens at the end of the frame and not immediately
// when calling the delete method.
//
world.delete_entity(entity).unwrap();
world.delete_entities(entities_slice).expect("Failed to delete entities from specified list.");
world.delete_all();

// Check if the entities has not been deleted.
//
let is_alive = world.is_alive(entity);
```

Components handling:

```rust
// Requires the entity!

let storage: ReadStorage<_> = world.read_storage::<MyComponent>();
let component: Option<&_> = storage.get(entity);

let mut storage: WriteStorage = world.write_storage::<MyComponent>();
let mut component: Option<&mut _> = storage.get_mut(entity);
```

### Systems

Systems represent the game logic; a system has a function that is invoked for every game loop.

```rust
struct MySystem;

impl<'a> System<'a> for MySystem {
  type SystemData = Read<'a, Time>;

  fn run(&mut self, data: Self::SystemData) {
    let delta: amethyst::core::timing::Time = data.delta_seconds();
  }
}
```

The SystemData associated type determines the what the system accesses (for each `Read` there is a `Write`):

- `Read<'a, Resource>`: get a reference to a resource of the type you specify; returns `Default::default()` for the type, if not found
- `ReadExpectExpect<'a, Resource>`: like `Read`, for resources that don't implement `default()` (will fail if not found)
- `ReadStorage<'a, Component>`: get reference to storage for the Component
- `Entities<'a>` create or destroy entities
- empty tuple

```rust
struct WalkPlayerUp {
  player: Entity,
}

impl<'a> System<'a> for WalkPlayerUp {
  type SystemData = WriteStorage<'a, Transform>;

  // Move the player by 0.1 y unit on each loop
  fn run(&mut self, mut transforms: Self::SystemData) {
    transforms.get_mut(self.player).unwrap().prepend_translation_y(0.1);
  }
}
```

(follow up: manipulating storages)
