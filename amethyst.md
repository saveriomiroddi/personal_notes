# Amethyst

- [Amethyst](#amethyst)
  - [Build book](#build-book)
  - [Concepts](#concepts)
    - [States](#states)
    - [Game data](#game-data)
    - [State/date example](#statedate-example)
  - [ECS](#ecs)

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

## ECS

Entities are trivial: `struct Entity(u32, Generation);`

Say, we have:

- `Person`: `(x, y)`, `name`
- `Bottle`: `(x, y)`, `shape`

The properties (of all the objects) could be stored in the three storages:

- `PositionComponent`: `(x,y)`
- `PersonComponent`: `name`
- `BottleComponent`: `shape`

This is very similar to an RDBMS (joins are indeed performed).
