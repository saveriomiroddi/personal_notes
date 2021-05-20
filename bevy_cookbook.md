# Bevy cookbook

- [Bevy cookbook](#bevy-cookbook)
  - [Hello world](#hello-world)
  - [Quit the app](#quit-the-app)
  - [Set the background color](#set-the-background-color)
  - [Display the framerate](#display-the-framerate)
  - [Grab the mouse](#grab-the-mouse)
  - [Track Assets Loading](#track-assets-loading)
  - [List All Resource Types](#list-all-resource-types)
  - [Convert cursor to world coordinates](#convert-cursor-to-world-coordinates)
  - [Custom camera projection](#custom-camera-projection)
  - [Pan+Orbit camera](#panorbit-camera)

## Hello world

```rust
use bevy::prelude::*;

struct Person;
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
}

struct GreetTimer(Timer);

// Use a timed greeter; without a timer, this is executed continuously (see DefaultPlugins note).
//
fn greet_people_timed(time: Res<Time>, mut timer: ResMut<GreetTimer>, query: Query<&Name, With<Person>>) {
  if timer.0.tick(time.delta()).just_finished() {
    for name in query.iter() {
      println!("Hello {}!", name.0);
    }
  }
}

pub struct HelloPlugin;

impl Plugin for HelloPlugin {
  fn build(&self, app: &mut AppBuilder) {
    app.insert_resource(GreetTimer(Timer::from_seconds(2.0, true)))
      .add_startup_system(add_people.system())
      .add_system(greet_people_timed.system());
  }
}

App::build()
  // WindowDescriptor stores the window properties.
  //
  .insert_resource(WindowDescriptor { title: "Hello W!".to_string(), width: 1280., height: 720., ..Default::default() })
  // The DefaultPlugins group includes the basic features to run a game; it makes a window popup because
  // it includes WindowPlugin and WinitPlugin. Since it also includes an event loop, the systems will
  // run in an infinite loop.
  //
  .add_plugins(DefaultPlugins)
  // In this plugin, we bundle the main setup.
  //
  .add_plugin(HelloPlugin)
  .run();
```

## Quit the app

```rust
use bevy::app::AppExit;

fn exit_system(mut exit: EventWriter<AppExit>) {
  exit.send(AppExit);
}

// For prototyping, Bevy provides a system, to exit on pressing the Esc key:
//
App::build().add_system(bevy::input::system::exit_on_esc_system.system())
```

## Set the background color

```rust
App::build().insert_resource(ClearColor(Color::rgb(0.4, 0.4, 0.4)))
```

## Display the framerate

```rust
use bevy::diagnostic::{FrameTimeDiagnosticsPlugin, LogDiagnosticsPlugin};

App::build()
  .add_plugin(LogDiagnosticsPlugin::default())
  .add_plugin(FrameTimeDiagnosticsPlugin::default())
```

## Grab the mouse

See https://bevy-cheatbook.github.io/cookbook/mouse-grab.html.

## Track Assets Loading

```rust
#[derive(Default)]
struct AssetsLoading(Vec<HandleId>);

fn setup(server: Res<AssetServer>, mut loading: ResMut<AssetsLoading>) {
    let font: Handle<Font> = server.load("my_font.ttf");
    let menu_bg: Handle<Texture> = server.load("menu.png");
    loading.0.push(font.id);
    loading.0.push(menu_bg.id);
}

fn check_assets_ready(server: Res<AssetServer>, loading: Res<AssetsLoading>) {
    match server.get_group_load_state(loading.0.iter().copied()) {
        LoadState::Failed => { /* one of our assets had an error */ }
        LoadState::Loaded => {}
        _                 => { return; /* NotLoaded/Loading: not fully ready yet */ }
    }

    // all assets are now ready
}
```

## List All Resource Types

List all the types that have been added (by the dev, not by Bevy) as resources:

```rust
fn print_resources(archetypes: &Archetypes, components: &Components) {
  // `get_short_name()` removes the path information, e.g `bevy_audio::audio::Audio` -> `Audio`;
  // in order to see the path, just use `info.name()`.
  let mut resources = archetypes
    .resource()
    .components()
    .map(|id| components.get_info(id).unwrap())
    .map(|info| TypeRegistration::get_short_name(info.name()))
    .collect::<Vec<_>>();

  resources.sort();
  resources.iter().for_each(|name| println!("{}", name));
}
```

## Convert cursor to world coordinates

See https://bevy-cheatbook.github.io/cookbook/cursor2world.html#convert-cursor-to-world-coordinates.

## Custom camera projection

See https://bevy-cheatbook.github.io/cookbook/custom-projection.html.

## Pan+Orbit camera

See https://bevy-cheatbook.github.io/cookbook/pan-orbit-camera.html.
