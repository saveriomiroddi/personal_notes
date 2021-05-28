# ggez

- [ggez](#ggez)
  - [Hello world](#hello-world)
  - [General design](#general-design)
  - [Draw](#draw)
    - [Viewport](#viewport)
    - [Drawables](#drawables)
      - [Text](#text)
      - [Meshes (Geometric shapes)](#meshes-geometric-shapes)
      - [Images](#images)
      - [Sprite batch](#sprite-batch)
  - [Audio](#audio)
  - [Timing](#timing)
  - [Events Handling](#events-handling)
    - [Input](#input)
  - [Misc](#misc)
    - [System](#system)

## Hello world

```rust
use ggez::event;
use ggez::graphics;
use ggez::{Context, GameResult};
use glam::*;

struct MainState {
    objects_x: f32,
    image: graphics::Image,
}

impl MainState {
    fn new(ctx: &mut Context) -> GameResult<MainState> {
        // Objects can also be instantiated inside draw(), where `&mut Context` is also available.

        // Linear filter is applied if not specified.
        //
        let mut image = graphics::Image::new(ctx, "/shot.png")?;
        image.set_filter(graphics::FilterMode::Nearest);

        Ok(MainState { objects_x: 0.0, image })
    }
}

impl event::EventHandler for MainState {
    fn update(&mut self, _ctx: &mut Context) -> GameResult {
        self.objects_x = self.objects_x % 800.0 + 1.0;

        Ok(())
    }

    fn draw(&mut self, ctx: &mut Context) -> GameResult {
        // Clear the screen with the given (background) color.
        graphics::clear(ctx, [0.1, 0.2, 0.3, 1.0].into());

        // (0, 0) is the top left.

        // Simple drawing, without transformations.
        //
        graphics::draw(ctx, &self.image, (Vec2::new(self.objects_x, 0.),))?;

        // Drawing with transformations.
        // Linear scaling is applied by default.
        // If there are no transformations, and the meshes have a position, one can pass `DrawParam::new()`
        // bare.
        //
        graphics::draw(
            ctx,
            &self.image,
            graphics::DrawParam::new()
                .dest(Vec2::new(self.objects_x, 0.))
                .rotation(0.5)
                .scale(Vec2::new(10., 10.)),
        )?;

        // Tell the graphics system to actually put everything on the screen. Must be called at the
        // end of draw().
        graphics::present(ctx)?;

        Ok(())
    }
}

pub fn main() -> GameResult {
    let (mut ctx, event_loop) = ggez::ContextBuilder::new("game_id", "author")
        .window_setup(ggez::conf::WindowSetup::default().title("Snake!"))
        .window_mode(ggez::conf::WindowMode::default().dimensions(800, 600))
        .add_resource_path("./resources")
        .build()?;
    let state = MainState::new(&mut ctx)?;
    event::run(ctx, event_loop, state)
}
```

## General design

Generally speaking:

- `MainState` is the world state
  - it includes the entities
- Both `MainState` and the entities include `update()` and `draw()`
  - when the methods are called on `MainState`, it calls them on the entities

## Draw

### Viewport

When setting a reversed viewport, it seems that the reverse coordinates are relative:

```rust
// This is an upside-down 800x600 viewport; note the second Y value.
//
let viewport = graphics::Rect::new(0.0, 600.0, 800.0, -600.0);
graphics::set_screen_coordinates(ctx, viewport).unwrap();
```

### Drawables

#### Text

```rust
// Coordinates: top left.
//
let font = graphics::Font::new(ctx, "/LiberationMono-Regular.ttf")?;
let text = graphics::Text::new(("Hello world!", font, 48.0));
```

#### Meshes (Geometric shapes)

```rust
// The type is `Mesh` for all.

// Coordinates: center.
//
graphics::Mesh::new_circle(
    ctx,
    graphics::DrawMode::fill(),
    Vec2::new(0.0, 0.0),
    100.0,
    2.0,
    Color::WHITE,
)?;

// Filled rectangle.
//
graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::fill(),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    Color::WHITE,
)?;

// Stroked rectangle (with thick borders).
// Drawing this on top of the previous yields a filled rectangle with thick borders.
//
let stroked_rect = graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::stroke(1.0),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    graphics::Color::new(1.0, 0.0, 0.0, 1.0),
)?;
```

Compose meshes, via `MeshBuilder`:

```rust
// Meshes can also be added by calling the methods individually on the MeshBuilder instance.

let mesh = graphics::MeshBuilder::new()
    .line(
        &[Vec2::new(200.0, 200.0), Vec2::new(400.0, 200.0), Vec2::new(400.0, 400.0)], // points
        4.0, // width
        Color::new(1.0, 0.0, 0.0, 1.0),
    )?
    .circle(
        DrawMode::fill(),
        Vec2::new(600.0, 380.0),
        40.0, 1.0, // radius, tolerance
        Color::new(1.0, 0.0, 1.0, 1.0),
    )?
    .build(ctx)?;
```

Perform batch instantiation and modification, via `MeshBatch`:

```rust
// In new() ////////////////////////////////////////////////////////////////////

let mesh = graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::fill(),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    Color::WHITE,
)?;

let mut mesh_batch = graphics::MeshBatch::new(mesh)?;

for x in 0..100 {
    mesh_batch.add(
        graphics::DrawParam::new()
            .dest(Vec2::new(x * 16.0, 100))
            .rotation(x * PI),
    );
}

// In update() /////////////////////////////////////////////////////////////////

let instances: &mut[DrawParam] = self.mesh_batch.get_instance_params_mut();

for instance in instances.iter_mut().take(50) {
    if let graphics::Transform::Values { ref mut rotation, .. } = instance.trans {
        *rotation += 0.001 * PI * delta_time;
    }
}

// In draw() ///////////////////////////////////////////////////////////////////

// Must flush before drawing! We modified 50, so we flush that number, starting from index 0.
//
self.mesh_batch.flush_range(ctx, graphics::MeshIdx(0), 50)?;
self.mesh_batch.draw(ctx, graphics::DrawParam::default())?;
```

Textured triangle:

```rust
let mb = &mut graphics::MeshBuilder::new();
let triangle_verts = vec![
    graphics::Vertex {
        pos: [100.0, 100.0],
        uv: [1.0, 1.0],
        color: [1.0, 0.0, 0.0, 1.0],
    },
    graphics::Vertex {
        pos: [0.0, 100.0],
        uv: [0.0, 1.0],
        color: [0.0, 1.0, 0.0, 1.0],
    },
    graphics::Vertex {
        pos: [0.0, 0.0],
        uv: [0.0, 0.0],
        color: [0.0, 0.0, 1.0, 1.0],
    },
];

let triangle_indices = vec![0, 1, 2];

let image = graphics::Image::new(ctx, "/rock.png")?;
mb.raw(&triangle_verts, &triangle_indices, Some(image))?;
mb.build(ctx)
```

#### Images

Coordinates: top left.

See [Hello world](#hello-world).

#### Sprite batch

Draw images in batch - for large amounts, it saves considerable time (bunnymark (1000 images) is ~8x as fast).

```rust
// In new() ////////////////////////////////////////////////////////////////////

let bunnybatch = SpriteBatch::new(image);

// In draw() ///////////////////////////////////////////////////////////////////

self.bunnybatch.clear();

for bunny in &self.bunnies {
    self.bunnybatch.add((bunny.position,));
}

graphics::draw(ctx, &self.bunnybatch, (Vec2::new(0.0, 0.0),))?;
```

## Audio

```rust
let shot_sound = audio::Source::new(ctx, "/pew.ogg")?;
```

## Timing

The internal timer is in the `TimeContext` type, which includes a `residual_update_dt`.

```rust
// Returns true if the time elapsed since the last check is higher than the target FPS.
// If more than a frame passed, is subtracts one frame time from `residual_update_dt`.
//
// This pattern is used to run the intended number of updates per second: if the time is early, no cycles
// are run, otherwise, one cycle is performed for each cycle fitting in the time since the last update.
//
while timer::check_update_time(ctx, UPDATES_PER_SECOND) {
    self.rotation += 0.01;
}

// The game doesn't implement a frame limiter; it requires yielding (via framework, although it's just
// forwarded to the O/S).
//
fn draw(&mut self, ctx: &mut Context) -> GameResult {
  // blah
  timer::yield_now();
  Ok()
}
```

## Events Handling

### Input

Example for a game where keeping a key pressed is meaningful; in a game where this doesn't apply, `key_up_event()` can be ignored.

```rust
impl EventHandler for MainState {
  fn key_down_event(&mut self, _ctx: &mut Context, keycode: KeyCode, _keymod: KeyMods, _repeat: bool) {
      match keycode {
          KeyCode::Up => { self.input.yaxis = 1.0; }
          KeyCode::Left => { self.input.xaxis = -1.0; }
          KeyCode::Right => { self.input.xaxis = 1.0; }
          KeyCode::Space => { self.input.fire = true; }
          KeyCode::Escape => event::quit(ctx),            // Exit API
          _ => {}
      }
  }

  fn key_up_event(&mut self, _ctx: &mut Context, keycode: KeyCode, _keymod: KeyMods) {
      match keycode {
          KeyCode::Up => { self.input.yaxis = 0.0; }
          KeyCode::Left | KeyCode::Right => { self.input.xaxis = 0.0; }
          KeyCode::Space => { self.input.fire = false; }
          _ => {}
      }
  }
}
```

Mouse:

```rust
fn mouse_button_down_event(&mut self,_ctx: &mut Context,button: input::mouse::MouseButton,_x: f32,_y: f32) {
    if button == input::mouse::MouseButton::Left { /* .. */ }
}
```

## Misc

### System

```rust
// Graphics system info (drivers etc.).
//
let info: String = graphics::renderer_info(&ctx)?;

// Canvas dimensions.
//
let (width, height) = graphics::drawable_size(ctx);
```
