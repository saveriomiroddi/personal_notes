# Bevy

- [Bevy](#bevy)
  - [Setup](#setup)
  - [Hello world](#hello-world)
  - [Architecture parts](#architecture-parts)
    - [Components/bundles](#componentsbundles)
    - [Resources](#resources)
    - [Systems](#systems)
      - [Events](#events)
      - [System Chaining](#system-chaining)
    - [Queries](#queries)
      - [Change detection](#change-detection)
    - [Commands](#commands)
      - [Recursive entities](#recursive-entities)
    - [Plugins](#plugins)
    - [States](#states)
    - [Stages](#stages)
    - [Run criteria](#run-criteria)
  - [Features](#features)
    - [Transforms](#transforms)
    - [Assets](#assets)
      - [Custom asset events/hooks](#custom-asset-eventshooks)
      - [Textures/sprites](#texturessprites)
    - [Input handling](#input-handling)
      - [Keyboard](#keyboard)
      - [Mouse](#mouse)
      - [Gamepad](#gamepad)
      - [Touchscreen](#touchscreen)
    - [Fixed timestep](#fixed-timestep)
    - [Time](#time)
    - [UI Layout](#ui-layout)
  - [Math APIs](#math-apis)

## Setup

```toml
bevy = {
  version = "0.5.0",
  features = [
    # Increase compilation speed by using dynamic linking, but disable for release!
    #
    "dynamic",
    # Enable if required; by default, only png/hdr/mp3 formats are supported.
    #
    "jpeg", "tga", "bmp", "dds", "flac", "ogg", "wav"
  ]
}
```

## Hello world

See the [cookbook](bevy_cookbook.md).

## Architecture parts

### Components/bundles

Components:

```rust
// ECS design: Don't add Xp as a field to Player - decouple it, so that it can be reused!
// Use wrapper structs for simple data types ("newtype" pattern)
//
struct PlayerXp(u32);

// Use empty structs to mark entities (and query them)
//
struct Enemy;
```

Component Bundles are like templates for entities with common components:

```rust
// WATCH OUT! Don't forget the Bundle attribute, otherwise, if a bundle is accidentally used as component,
// no error is raised.
//
#[derive(Bundle)]
struct PlayerBundle {
  health: Health,
  _p: Player,

  // Bundles can be nested
  #[bundle]
  sprite: SpriteSheetBundle,
}

// Tuples of components are considered bundles
//
(ComponentA, ComponentB, ComponentC)
```

### Resources

Resources can be global or local; try to use globals as little as possible (components make a better design, as they can be shared).  
Any type (struct/enum) can be used.

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

Usage:

```rust
// Global (shared).
//
fn complex_system(res: ResMut<ResourceA>) { /* ... */ }

// Local - different systems will access a different instance.
//
fn my_system1(mut local: Local<MyState>) { /* ... */ }
fn my_system2(mut local: Local<MyState>) { /* ... */ }
```

### Systems

Systems are functions executed by the engine. Access (R/W) to the world from a given system is determined via function parameters.  
Don't forget that they run concurrently!

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
```

Bevy handles resources contention by looking at the system signatures.

It's possible to specify the systems ordering by using labels.

```rust
App::build()
  .add_system_set(
    SystemSet::new()
      .label("input")
      .with_system(keyboard_input.system())
      .with_system(gamepad_input.system())
  )

  .add_system(update_map.system().label("map"))



#[derive(Debug, Clone, PartialEq, Eq, Hash)]
#[derive(SystemLabel)]
enum MySystems { InputSet, Movement }

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
#[derive(StageLabel)]
enum MyStages { Prepare, Cleanup }

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
#[derive(StageLabel)]
struct DebugStage;

App::build()
  // A group ("set") of systems
  //
  .add_system_set(
    SystemSet::new()
      .label(MySystems::InputSet)
      .with_system(keyboard_input.system())
      .with_system(gamepad_input.system())
  )

  // Strings can be used, but type system advantages are lost.
  //
  .add_system(debug_movement.system().label("temp-debug"))

  .add_system(input_parameters.system().before("input"))

  // before() and after() can be used on the same system.
  //
  .add_system(
    player_movement.system()
      .label("player_movement")
      .after("input")
      .after("map")
  )
  // Add custom stages. `CoreStage` is a Bevy enum.
  //
  .add_stage_before(CoreStage::Update, MyStages::Prepare, SystemStage::parallel())
  .add_stage_after(CoreStage::Update, DebugStage, SystemStage::parallel())
```

Labels and labelled items are M:N.

#### Events

Systems can communicate via a message queue - the `Event`s system.

WATCH OUT!! Events persists only until the end of the next frame (= max 2 frames).  
Additionally, due to the async nature, a system may receive the event only on the next frame ("1-frame-lag" problem); if this is a problem, must explicitly order systems.

```rust
struct LevelUpEvent(Entity);

// Generate an event, via EventWriter<T>.
//
fn player_level_up(mut levelup: EventWriter<LevelUpEvent>, query: Query<(Entity, &PlayerXp)>) {
  for (entity, xp) in query.iter() {
    if xp.0 > 1000 { levelup.send(LevelUpEvent(entity)); }
  }
}

// Read an event, via EventReader<T>.
//
fn debug_levelups(mut levelup: EventReader<LevelUpEvent>) {
  for ev in levelup.iter() {
    eprintln!("Entity {:?} leveled up!", ev.0);
  }
}

// Don't forget to register (custom) events!
//
App::build().add_event::<LevelUpEvent>()
```

#### System Chaining

Systems can be chained, so that the output of one is the input of the next.

```rust
fn net_receive(mut netcode: ResMut<MyNetProto>) -> std::io::Result<()> {
  netcode.receive_updates()?;
  Ok(())
}

fn handle_io_errors(In(result): In<std::io::Result<()>>) {
  if let Err(e) = result { eprintln!("I/O error occurred: {}", e); }
}

// WATCH OUT! Chaining must be registered:
//
App::build().add_system(net_receive.system().chain(handle_io_errors.system()))
```

WATCH OUT!

- for passing data around, it's best to use Events;
- coupling systems by chaining them may limit the parallelization;
- if a system requires wide mutable access, it may limit the parallelization.

### Queries

Used to access entity components. WATCH OUT! Bundles must be queried via component(s), not bundle type!

```rust
fn check_zero_health(
  // Find entities that have (`Health` & `Transform`) components, and are optionally a `Player`.
  // Entity is the entity id (not required).
  // With/out are (optional) filters: the entities must include an Ally, but not an Enemy.
  // For an Or filter example, see [Change detection](#change-detection).
  mut query: Query<
    (Entity, &Health, &mut Transform, Option<&Player>),
    (With<Ally>, Without<Enemy>)
  >
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

When it's not possible for systems to query due to mutability conflicts, query sets can be used; they prevent conflicting resources to be accessed at the same time.  
A set can have up to 4 queries.

```rust
fn reset_health(
  // access the health of enemies and the health of players (some entities could be both!)
  mut q: QuerySet<(
    Query<&mut Health, With<Enemy>>,
    Query<&mut Health, With<Player>>
  )>,
) {
  for mut health in q.q0_mut().iter_mut() { /* ... */ }
  for mut health in q.q1_mut().iter_mut() { /* ... */ }
}
```

#### Change detection

Use query filters:

- `Added<T>`: component added to an entity, or entity with it created
- `Changed<T>`: some as `Added` or component mutably accessed (`DerefMut` access, not necessarily changed; not query only)

If triggering `Changed` for unchanged variables is undesirable, modify the code upstream to set a variable (`DerefMut`) only if the value changed.

```rust
// Print the stats of friendly players when they change:
//
fn debug_stats_change(
  query: Query<
    (&Health, &PlayerXp),
    (Without<Enemy>, Or<(Changed<Health>, Changed<PlayerXp>)>),
  >,
) {
  for (health, xp) in query.iter() {
    eprintln!("hp: {}+{}, xp: {}", health.hp, health.extra, xp.0);
  }
}

// Detect new enemies and print their health:
//
fn debug_new_hostiles(query: Query<(Entity, &Health), Added<Enemy>>) {
  for (entity, health) in query.iter() {
    eprintln!("Entity {:?} is now an enemy! HP: {}", entity, health.hp);
  }
}
```

WATCH OUT 1-frame lag; for `Added`, [stages](#stages) may be required.

### Commands

Commands add/remove/update stuff"

```rust
fn spawn_player(mut commands: Commands) {
  // Manage resources
  //
  commands.insert_resource(GoalsReached { main_goal: false, bonus: false });
  commands.remove_resource::<MyResource>();

  // Create a new entity using `spawn`
  // WATCH OUT! insert_bundle() adds all the components of a bundle, while insert(Bundle) inserts the
  // bundle as a single component!
  //
  let entity_id = commands.spawn()
    .insert(ComponentA)                 // add a component
    .insert_bundle(MyBundle::default()) // add a bundle
    .id();                              // get the Entity ID

  // Shorthand for creating an entity with a bundle.
  //
  commands.spawn_bundle(PlayerBundle {
    name: PlayerName("Henry".into()),
    _p: Player,
  });

  // Since tuples are bundles, they can be used with spawn_bundle().
  //
  let other = commands.spawn_bundle((
    ComponentA::default(),
    ComponentB::default(),
  )).id();

  // Add/remove components of an existing entity.
  //
  commands.entity(entity_id)
    .insert(ComponentB)
    .remove::<ComponentA>()
    .remove_bundle::<MyBundle>();

  // Despawn an entity.
  //
  commands.entity(other).despawn();
}

fn make_all_players_hostile(mut commands: Commands, query: Query<Entity, With<Player>>) {
  for entity in query.iter() {
    commands.entity(entity).insert(Enemy); // add an `Enemy` component to the entity
  }
}
```

#### Recursive entities

A hierarchy can be established among entities:

```rust
commands.entity(parent).push_children(&[child]);

// Alternative:
//
commands.spawn_bundle(MyParentBundle::default())
  .with_children(|parent| {
    parent.spawn_bundle(MyChildBundle::default());
  });

// Despawn a whole hierarchy:
//
commands.entity(entity).despawn_recursive();
```

Querying; use the `Parent`/`Children` reserved components:

```rust
// Query parent, from children:
//
fn camera_with_parent(q_child: Query<(&Parent, &Transform), With<Camera>>, q_parent: Query<&GlobalTransform>) {
  for (parent, child_transform) in q_child.iter() {
    // `parent` contains the Entity ID we can use to query components from the parent:
    //
    let parent_global_transform = q_parent.get(parent.0);
  }
}

// Query children, from parent:
//
fn process_squad_damage(q_parent: Query<(&MySquadDamage, &Children)>, q_child: Query<&MyUnitHealth>) {
  for (squad_dmg, children) in q_parent.iter() {
    // `children` is a collection of Entity IDs:
    //
    for &child in children.iter() {
      let health = q_child.get(child);
    }
  }
}
```

For transforms, there is special support.

### Plugins

Plugins are convenient to group things to add to the app builder, e.g. physics, input handling etc.:

```rust
pub struct HelloPlugin;

impl Plugin for HelloPlugin {
  fn build(&self, app: &mut AppBuilder) {
    app.insert_resource(GreetTimer(Timer::from_seconds(2.0, true)))
      .add_startup_system(add_people.system())
      .add_system(greet_people_timed.system());
  }
}
```

Plugins can be grouped:

```rust
struct MyPluginGroup;

impl PluginGroup for MyPluginGroup {
  fn build(&mut self, group: &mut PluginGroupBuilder) {
    group
      .add(FooPlugin)
      .add(BarPlugin);
  }
}
```

It's possible to register plugin groups while excluding certain plugins:

```ruby
App::build()
  .add_plugins_with(DefaultPlugins, |plugins| {
    plugins.disable::<LogPlugin>()
  })
```

### States

States are the global app states.  
It's convenient to use plugins to group systems to use with states.

WATCH OUT! Since states aren internally implemented via run criteria, they will conflict with other run criteria uses, e.g. fixed timestep.

Setup:

```rust
#[derive(Debug, Clone, Eq, PartialEq, Hash)]
enum AppState { MainMenu, InGame, Paused }

App::build()
  // add the app state type
  .add_state(AppState::MainMenu)

  // add systems to run regardless of state, as usual
  .add_system(play_music.system())

  // systems to run only in the main menu
  .add_system_set(
    SystemSet::on_update(AppState::MainMenu)
      .with_system(handle_ui_buttons.system())
  )

  // setup when entering the state
  .add_system_set(
    SystemSet::on_enter(AppState::MainMenu)
      .with_system(setup_menu.system())
  )

  // cleanup when exiting the state
  .add_system_set(
    SystemSet::on_exit(AppState::MainMenu)
      .with_system(close_menu.system())
  )
```

Match:

```rust
fn play_music(app_state: Res<State<AppState>>) {
  match app_state.current() {
    AppState::MainMenu => { /* ... */ }
    AppState::InGame => { /* ... */ }
    AppState::Paused => { /* ... */ }
  }
}
```

Stack:

```rust
App::build()
  // player movement only when actively playing
  .add_system_set(
    SystemSet::on_update(AppState::InGame)
      .with_system(player_movement.system())
  )
  // player idle animation while paused
  .add_system_set(
    SystemSet::on_inactive_update(AppState::InGame)
      .with_system(player_idle.system())
  )
  // animations both while paused and while active
  .add_system_set(
    SystemSet::on_in_stack_update(AppState::InGame)
      .with_system(animate_trees.system())
      .with_system(animate_water.system())
  )
  // things to do when becoming inactive
  .add_system_set(
    SystemSet::on_pause(AppState::InGame)
      .with_system(hide_enemies.system())
  )
  // things to do when becoming active again
  .add_system_set(
    SystemSet::on_resume(AppState::InGame)
      .with_system(reset_player.system())
  )
  // setup when first entering the game
  .add_system_set(
    SystemSet::on_enter(AppState::InGame)
      .with_system(setup_player.system())
      .with_system(setup_map.system())
  )
  // cleanup when finally exiting the game
  .add_system_set(
    SystemSet::on_exit(AppState::InGame)
      .with_system(despawn_player.system())
      .with_system(despawn_map.system())
  )
```

Change; all the current state's systems complete before moving to the next system (there's no limit to the number of state changes within a frame):

```rust
// Switch via stack.
// Go into the pause screen, and back into the game.
//
app_state.push(AppState::Paused).unwrap();
app_state.pop().unwrap();

// Direct set.
//
fn enter_game(mut app_state: ResMut<State<AppState>>) {
  app_state.set(AppState::InGame).unwrap();
  // ^ this can fail if we are already in the target state
  // or if another state change is already queued
}
```

WATCH OUT!! Key-initiated state changes require Input to be reset:

```rust
fn esc_to_menu(mut keys: ResMut<Input<KeyCode>>, mut app_state: ResMut<State<AppState>>) {
  if keys.just_pressed(KeyCode::Escape) {
    app_state.set(AppState::MainMenu).unwrap();
    keys.reset(KeyCode::Escape);
  }
}
```

### Stages

Each frame goes through stages; Bevy has 5: `First`, `PreUpdate`, `Update`, `PostUpdate`, `Last`.

By default, user-defined systems are placed in `Update`; if systems are added to the other stages, beware of interactions with Bevy's internal ones.

See https://bevy-cheatbook.github.io/programming/stages.html.

### Run criteria

Run criteria allow low-level systems running specification. See: https://bevy-cheatbook.github.io/programming/run-criteria.html.

## Features

### Transforms

Bevy's coordinates are left-hand.

In 2D:

- (0, 0) is at the center of the screen;
- by default, the camera is far (999.9, slightly before Z-clipping), so that the sprites can be displayed, however, in case of custom transforms, this must be take into account;
- it's convenient to put the background on z=0, and other sprites with increasing z (based on priority).

There are two transforms: `Transform` (relative to parent) and `GlobalTransform` (best to use read-only; if a component has no parent, T = GT)

WATCH OUT! GT is synced with T in the `PostUpdate` stage, so be careful.

### Assets

Assets are loaded asynchronously, but the handle is returned (and can be used) immediately.
Asset changes are detected by Bevy, and hot-reloaded; this works fine for simple parts, but more complex ones (e.g. scene files) may require custom reloading logic (via `AssetEvent`).

Load/access assets:

```rust
struct UiFont(Handle<Font>);

struct SpriteSheets {
  map_tiles: Handle<TextureAtlas>,
}

struct ExtraAssets(Vec<HandleUntyped>);

fn load_ui_font(mut commands: Commands, server: Res<AssetServer>) {
  // Multiple `load`s of the same file return the same handle.
  //
  let handle: Handle<Font> = server.load("font.ttf");

  // Store it - avoids collection and consequent unloading.
  //
  commands.insert_resource(UiFont(handle));
}

fn use_sprites(handles: Res<SpriteSheets>, atlases: Res<Assets<TextureAtlas>>, textures: Res<Assets<Texture>>) {
  // The two getters can return None if the assets are not loaded yet.

  if let Some(atlas) = atlases.get(&handles.map_tiles) { /* ... */ }
  if let Some(map_tex) = textures.get("map.png") { /* ... */ }
}

fn paths_and_clones(mut commands: Commands, server: Res<AssetServer>) {
  // Load with "Asset path".
  //
  let mesh: Handle<Mesh> = server.load("scene.gltf#Mesh0/Primitive0");

  // Strong and weak clones (have standard strong/weak referencing behavior).
  //
  let strong_h_clone = mesh.clone();
  let weak_h_clone = mesh.clone_weak();

  // Untyped clones allow storing mixed-typed collections.
  //
  let untyped_clone = my_collection.clone_untyped();

  // Untyped loading! Formats are detected by extension.
  //
  if let Ok(handles) = server.load_folder("extra") {
    commands.insert_resource(ExtraAssets(handles));
  }
}
```

Manual assets generation (e.g. for procedural generation):

```rust
fn add_material(mut materials: ResMut<Assets<StandardMaterial>>) {
  let new_mat = StandardMaterial { base_color: Color::rgba(0.25, 0.50, 0.75, 1.0), unlit: true, ..Default::default() };
  materials.add(new_mat);
}
```

#### Custom asset events/hooks

See https://bevy-cheatbook.github.io/features/assets.html#assetevent.

#### Textures/sprites

For textures/images, Y points downwards. This is consistent with the majority of the image formats, but inconsistent with the rest of Bevy (and OpenGL).

Use the sprite-flipping feature to invert this behavior:

```rust
commands.spawn_bundle(SpriteBundle {
  sprite: Sprite { flip_y: true, flip_x: false, ..Default::default() }, ..Default::default()
});
```

### Input handling

Input handling can be managed via resources (higher level) and events (lower level).

#### Keyboard

```rust
fn keyboard_input(keys: Res<Input<KeyCode>>) {
  if keys.just_pressed(KeyCode::Space) { /* Space was pressed */ }
  if keys.just_released(KeyCode::LControl) { /* Left Ctrl was released */ }
  if keys.pressed(KeyCode::W) { /* W is being held down */ }
}

fn keyboard_events(mut key_evr: EventReader<KeyboardInput>) {
  use bevy::input::ElementState;

  // Scan codes are provided, but currently not usable.
  //
  for ev in key_evr.iter() {
    match ev.state {
      ElementState::Pressed => println!(ev.key_code),
      ElementState::Released => println!(ev.key_code)
    }
  }
}
```

#### Mouse

Buttons/wheel:

```rust
fn mouse_button_input(buttons: Res<Input<MouseButton>>) {
  if buttons.just_pressed(MouseButton::Left) { /* Left button was pressed */ }
  if buttons.just_released(MouseButton::Left) { /* Left Button was released */ }
  if buttons.pressed(MouseButton::Right) { /* Right Button is being held down */ }
}

fn mouse_button_events(mut mousebtn_evr: EventReader<MouseButtonInput>) {
  use bevy::input::ElementState;

  for ev in mousebtn_evr.iter() {
    match ev.state {
      ElementState::Pressed => println!(ev.button),
      ElementState::Released => println!(ev.button),
    }
  }
}

fn scroll_events(mut scroll_evr: EventReader<MouseWheel>) {
  use bevy::input::mouse::MouseScrollUnit;

  for ev in scroll_evr.iter() {
    match ev.unit {
      MouseScrollUnit::Line => println!("Scroll (line units): vertical: {}, horizontal: {}", ev.y, ev.x),
      MouseScrollUnit::Pixel => println!("Scroll (pixel units): vertical: {}, horizontal: {}", ev.y, ev.x),
    }
  }
}
```

Relative motion:

```rust
fn mouse_motion(mut motion_evr: EventReader<MouseMotion>) {
  for ev in motion_evr.iter() {
    println!("Mouse moved: X: {} px, Y: {} px", ev.delta.x, ev.delta.y);
  }
}
```

Cursor:

```rust
fn cursor_position(windows: Res<Windows>) {
  // Games typically only have one window (the primary window).
  // For multi-window applications, you need to use a specific window ID here.
  let window = windows.get_primary().unwrap();

  if let Some(_position) = window.cursor_position() {
    // cursor is inside the window, position given
  } else {
    // cursor is not inside the window
  }
}

// To detect when the pointer is moved, use CursorMoved events to get the updated coordinates:

fn cursor_events(mut cursor_evr: EventReader<CursorMoved>) {
  for ev in cursor_evr.iter() {
    println!("New cursor position: X: {}, Y: {}, in Window ID: {:?}", ev.position.x, ev.position.y, ev.id);
  }
}
```

#### Gamepad

Backed by the [gilrs library](https://gitlab.com/gilrs-project/gilrs).

Connection:

```rust
// Simple resource to store the ID of the connected gamepad; we need to know which gamepad to use for player input.
//
struct MyGamepad(Gamepad);

fn gamepad_connections(mut commands: Commands, my_gamepad: Option<Res<MyGamepad>>, mut gamepad_evr: EventReader<GamepadEvent>) {
  for GamepadEvent(id, kind) in gamepad_evr.iter() {
    match kind {
      GamepadEventType::Connected => {
        // If we don't have any gamepad yet, use this one
        if my_gamepad.is_none() { commands.insert_resource(MyGamepad(*id)) }
      }
      GamepadEventType::Disconnected => {
        // If it's the one we previously associated with the player, disassociate it:
        if let Some(MyGamepad(old_id)) = my_gamepad.as_deref() {
          if old_id == id { commands.remove_resource::<MyGamepad>() }
        }
      }
      // other events are irrelevant
      _ => {}
    }
  }
}
```

Input:

```rust
fn gamepad_input(axes: Res<Axis<GamepadAxis>>, buttons: Res<Input<GamepadButton>>, my_gamepad: Option<Res<MyGamepad>>) {
  let gamepad_id = if let Some(gp) = my_gamepad { gp.0 } else { return };

  // The joysticks are represented using a separate axis for X and Y

  let axis_lx = GamepadAxis(gamepad_id, GamepadAxisType::LeftStickX);
  let axis_ly = GamepadAxis(gamepad_id, GamepadAxisType::LeftStickY);

  if let (Some(x), Some(y)) = (axes.get(axis_lx), axes.get(axis_ly)) {
    let left_stick_pos = Vec2::new(x, y); // for convenience

    // implement a dead-zone to ignore small inputs
    if left_stick_pos.length() > 0.1 { /* ... */ }
  }

  let jump_button = GamepadButton(gamepad_id, GamepadButtonType::South);
  let heal_button = GamepadButton(gamepad_id, GamepadButtonType::East);

  if buttons.just_pressed(jump_button) { /* button pressed: make the player jump */ }
  if buttons.pressed(heal_button) { /* button being held down: heal the player */ }
}
```

#### Touchscreen

See https://bevy-cheatbook.github.io/features/input-handling.html#touchscreen.

### Fixed timestep

Use to run systems at fixed interval.

```rust
use bevy::core::FixedTimestep;

App::build()
  .add_system_set(
    SystemSet::new()
      .with_run_criteria(FixedTimestep::step(1.0)) // 1 Hz
      .with_system(system_1.system())
  )
  .add_system_set(
    SystemSet::new()
      .with_run_criteria(FixedTimestep::step(0.5)) // 2 Hz
      .with_system(system_2.system())
  )
```
WATCH OUT!

- events persist 2 frames, so if the fixed timestep is too slow, it will miss them;
- timing is not exact, due to the inexact run criteria underlying nature;
- etc.etc.!.

See https://bevy-cheatbook.github.io/features/fixed-timestep.html (and referred examples) for more details.

### Time

Don't use `std::time::Instant::now()` to get the time - use `Res<Time>`: due to parallel execution, the former will be inconsistent/inaccurate (respect to the game time).

### UI Layout

Since the Y starts at the bottom, the UI flows from there; use `FlexDirection::ColumnReverse` to flow from the top.

## Math APIs

```rust
const_vec2!   // create a const Vec2; also for: Mat2/3/4, Quat, Vec3/3a/4
```
