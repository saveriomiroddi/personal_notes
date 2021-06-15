# ggez

- [ggez](#ggez)
  - [Hello world](#hello-world)
  - [General design](#general-design)
  - [Draw](#draw)
    - [Text](#text)
    - [Meshes (Geometric shapes)](#meshes-geometric-shapes)
    - [Images](#images)
    - [Sprites batching](#sprites-batching)
  - [Screen](#screen)
    - [Graphics configuration](#graphics-configuration)
    - [Viewport](#viewport)
    - [Canvas](#canvas)
  - [Audio/Sound](#audiosound)
  - [Timing](#timing)
  - [Events Handling](#events-handling)
    - [Input](#input)
    - [Window](#window)
  - [Misc](#misc)
    - [Graphics(-related) info](#graphics-related-info)
    - [Files](#files)
    - [Logging](#logging)
    - [Algebra](#algebra)

## Hello world

```rust
use ggez::event;
use ggez::graphics;
use ggez::{Context, GameResult};
use glam::Vec2;                  // Add `glam = {version = "...", features = ["mint"]}` dependency to the manifest

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
            DrawParam::new()
                .dest(Vec2::new(self.objects_x, 0.))
                .rotation(0.5)
                .scale(Vec2::new(10., 10.)),
        )?;

        // Tell the graphics system to actually put everything on the screen. Must be called at the
        // end of draw().
        graphics::present(ctx)?;

        timer::yield_now();

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
        DrawParam::new()
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
self.mesh_batch.draw(ctx, DrawParam::default())?;
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

### Sprites batching

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

## Screen

It's possible to draw to an arbitrary destination; see `render_to_image.rs`.

### Graphics configuration

Fullscreen/windowed, and resizability:

```rust
enum conf::FullscreenType {
    Windowed,
    True,     // also it allows us to set different resolutions.
    Desktop,  // modern preference; plays nicer with multiple monitors.
}

ggez::graphics::set_fullscreen(ctx, fullscreen_type)?;

context_builder
    .window_mode(
        conf::WindowMode::default()
            .fullscreen_type(conf::FullscreenType::Windowed)
            .vsync(true) // default: true
            .resizable(true)
    )
```

Antialiasing:

```rust
// Supported: 2⁰ to 2⁴
//
context_builder
    .window_setup(
        // or: `conf::NumSamples::Four`
        conf::WindowSetup::default().samples(conf::NumSamples::try_from(4)?)
    )
```

Backend:

```rust
enum Backend {
    // Defaults to 3.2, which is supported by basically every machine since 2009 or so
    OpenGL { major: u8, minor: u8 },
    // Defaults to 3.0, used for mobile devices; older versions have limitations.
    #[allow(clippy::upper_case_acronyms)]
    OpenGLES { major: u8, minor: u8 },
}

let backend = conf::Backend::default(); // OpenGL
let backend = conf::Backend::OpenGLES { major: 3, minor: 0 };

context_builder
    .backend(backend)
```

### Viewport

The `set_screen_coordinates()` function sets the viewport; this implies that if the window size is the same, and the viewport size increases, the image drawn gets smaller!

```rust
println!("Viewport size: {:?}", graphics::screen_coordinates(ctx));
```

It's also possible to draw upside down:

```rust
// This is an upside-down 800x600 viewport; note the second Y value.
//
let viewport = graphics::Rect::new(0.0, 600.0, 800.0, -600.0);
graphics::set_screen_coordinates(ctx, viewport).unwrap();
```

Since the event coordinates are absolute, if the coordinates system (viewport) is changed, the logical coordinates need to be computed:

```rust
let screen_rect = graphics::screen_coordinates(ctx);
let size = graphics::window(ctx).inner_size(); // seems the same as graphics::drawable_size(ctx)
logical_x = (x / (size.width  as f32)) * screen_rect.w + screen_rect.x;
logical_y = (y / (size.height as f32)) * screen_rect.h + screen_rect.y;
```

### Canvas

It's possible to draw to a canvas, and then to screen.

The example (`hello_canvas.rs`) is confusing - it seems that the screen canvas needs to be set to `None` in order for the canvas to be written to the screen, but it's written even after.

It is also [buggy](https://docs.rs/ggez/0.4.4/ggez/graphics/type.Canvas.html).

## Audio/Sound

Don't forget that the reference must be in scope, somewhere :)

```rust
let sound = audio::Source::new(ctx, "/pew.ogg")?;
sount.set_volume(0.5);

// Playback operations (it's correct that some methods require Context, while others don't)

sound.play_detached(ctx)?;     // Play asynchronously (even if dropped), without wait; can't be stopped
sound.play_later()?;           // Queue playback, waiting each time for the sound to finish
sound.play(ctx).unwrap();      // Shortcut for stop() + play_later()
sound.set_repeat(true);
sound.pause();
sound.resume();
sound.stop(ctx)?;

// Example queries

while sound.playing() { println!("Elapsed time: {:?}", sound.elapsed()) }
sound.stopped();
sound.volume();
sound.repeat();

// Example effects

sound.set_pitch(2.0);
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

    // Text input (simplified handling); called after (each) key_down event.
    //
    fn text_input_event(&mut self, _ctx: &mut Context, ch: char) { /* ... */ }
}

// Key status can be checked from the context:

let a_key_pressed: bool = keyboard::is_key_pressed(ctx, KeyCode::A);
let shift_key_pressed: bool =  keyboard::is_mod_active(ctx, keyboard::KeyMods::SHIFT)
println!("Pressed keys: {:?}", keyboard::pressed_keys(ctx));
let key_pressed_repeatedly: bool = keyboard::is_key_repeated(ctx, KeyCode::A);

let keys_pressed: &HashSet<KeyCode> = keyboard::pressed_keys(ctx);
```

Mouse:

```rust
fn mouse_button_down_event(&mut self, _ctx: &mut Context, button: input::mouse::MouseButton, x: f32, y: f32) {
    if button == input::mouse::MouseButton::Left { /* .. */ }
}

fn mouse_button_up_event(&mut self, _ctx: &mut Context, button: MouseButton, x: f32, y: f32) { /* ... */ }
fn mouse_motion_event(&mut self, _ctx: &mut Context, x: f32, y: f32, xrel: f32, yrel: f32) { /* ... */ }
fn mouse_wheel_event(&mut self, _ctx: &mut Context, x: f32, y: f32) { /* ... */ }
```

Pad:

```rust
fn gamepad_button_down_event(&mut self, _ctx: &mut Context, btn: Button, id: GamepadId) { /* .. */ }
fn gamepad_button_up_event(&mut self, _ctx: &mut Context, btn: Button, id: GamepadId) { /* .. */ }
fn gamepad_axis_event(&mut self, _ctx: &mut Context, axis: Axis, value: f32, id: GamepadId) { /* .. */ }
```

### Window

```rust
fn resize_event(&mut self, ctx: &mut Context, new_width: f32, new_height: f32) { /* ... */ }

fn focus_event(&mut self, _ctx: &mut Context, gained: bool) { /* gained/lost */ }
```

## Misc

### Graphics(-related) info

Window functions return zeros if the window doesn't exist.

```rust
// Graphics system info (drivers etc.).
//
let info: String = graphics::renderer_info(&ctx)?;

// Canvas dimensions.
//
let (width, height) = graphics::drawable_size(ctx);

// Size of the window, including borders, titlebar, etc.
//
let (width, height) = graphics::size(ctx);

// Averaged from the last 200 frames.
//
let fps: f64 = ggez::timer::fps(ctx);
```

### Files

The filesystem library implements a VFS.

```rust
// Print context filesystem info
filesystem::print_all(ctx);

// Get files of a dir
let files: Box<dyn Iterator<Item = path::PathBuf>> = filesystem::read_dir(ctx, "/")?;

let test_file = path::Path::new("/path/to/file");

// Write
{
    let mut file = filesystem::create(ctx, test_file)?;
    file.write_all(b"foo")?;
}

// Append
{
    let options = filesystem::OpenOptions::new().append(true);
    let mut file = filesystem::open_options(ctx, test_file, options)?;
    file.write_all(b"foo")?;
}

// Read
{
    let mut buffer = Vec::new();
    let mut file = filesystem::open(ctx, test_file)?;
    file.read_to_end(&mut buffer)?;
}

// Delete
filesystem::delete(ctx, "/path/to/file")?;

// Write a configuration file `conf.toml` to the user dir.
let config = conf::Conf::new();
filesystem::write_config(ctx, &config)?;

// Read a configuration file, named `conf.toml`, in any resource dir.
let config = filesystem::read_config(ctx, )?;
```

### Logging

See `logging.rs` example.

### Algebra

ggez uses the `glam` crate:

```rust
// Related types: Mat2/3/4, Quat, Vec3/4; Vec3A is SIMD

const X_AXIS: Vec2 = const_vec2!([1., 0.]);
let vec2 = glam::Vec2::new(200., 100.);
let normalize = vec2.normalize();
```
