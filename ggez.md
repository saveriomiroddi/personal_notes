# ggez

- [ggez](#ggez)
  - [Hello world](#hello-world)
  - [Drawables](#drawables)
    - [Text](#text)
    - [Meshes (Geometric shapes)](#meshes-geometric-shapes)
    - [Images](#images)
  - [Misc](#misc)
    - [Timing](#timing)
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
        .add_resource_path("./resources")
        .build()?;
    let state = MainState::new(&mut ctx)?;
    event::run(ctx, event_loop, state)
}
```

## Drawables

### Text

```rust
// Coordinates: top left.
//
let font = graphics::Font::new(ctx, "/LiberationMono-Regular.ttf")?;
let text = graphics::Text::new(("Hello world!", font, 48.0));
```

### Meshes (Geometric shapes)

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

Compose meshes:

```rust
let builder = &mut graphics::MeshBuilder::new();

builder.line(
    &[Vec2::new(200.0, 200.0), Vec2::new(400.0, 200.0), Vec2::new(400.0, 400.0)], // points
    4.0, // width
    Color::new(1.0, 0.0, 0.0, 1.0),
)?;

builder.ellipse(
    DrawMode::fill(),
    Vec2::new(600.0, 200.0),
    50.0, 120.0, 1.0 // radius1, radius2, tolerance
    Color::new(1.0, 1.0, 0.0, 1.0),
)?;

builder.circle(
    DrawMode::fill(),
    Vec2::new(600.0, 380.0),
    40.0, 1.0, // radius, tolerance
    Color::new(1.0, 0.0, 1.0, 1.0),
)?;

// Creates a Mesh instance.
//
builder.build(ctx)
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

### Images

Coordinates: top left.

See [Hello world](#hello-world).

## Misc

### Timing

The internal timer is in the `TimeContext` type, which includes a `residual_update_dt`.

```rust
// Returns true if the time elapsed since the last check is higher than the target FPS.
// If more than a frame passed, is subtracts one frame time from `residual_update_dt`.
//
// This example is a bit confusing, but what it does is to perform one rotation change for each frame.
// If the game is late, one cycle is performed for each frame of delay.
//
while timer::check_update_time(ctx, DESIRED_FPS) {
    self.rotation += 0.01;
}
```

### System

```rust
// Graphics system info (drivers etc.).
//
let info: String = graphics::renderer_info(&ctx)?;
```
