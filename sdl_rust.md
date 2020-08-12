# SDL/Rust
- [SDL/Rust](#sdlrust)
  - [Base Rust Programming By Example](#base-rust-programming-by-example)

## Base Rust Programming By Example

Create a window, color it red, and draw in it a red square.

```rust
let sdl_context = sdl2::init()?;

let video_subsystem = sdl_context.video()?;

// Parameters are: title, width, height
let window = video_subsystem
  .window("Tetris", 800, 600)
  .position_centered()
  .opengl()
  .build()?;

let mut canvas = window
  .into_canvas()
  .target_texture()
  .present_vsync() // Enable v-sync.
  .build()?;

let texture_creator = canvas.texture_creator();

let mut square_texture =
  texture_creator.create_texture_target(None, TEXTURE_SIZE, TEXTURE_SIZE)?;

canvas.with_texture_canvas(&mut square_texture, |texture| {
  texture.set_draw_color(Color::RGB(0, 255, 0));
  texture.clear();
})?;

let mut event_pump = sdl_context.event_pump()?;

'main_loop: loop {
  for event in event_pump.poll_iter() {
    match event {
      Event::Quit { .. }
      | Event::KeyDown {
        keycode: Some(Keycode::Escape),
        ..
      } => {
        break 'main_loop;
      }
      _ => {}
    }
  }

  canvas.set_draw_color(Color::RGB(255, 0, 0));
  // Draw the window.
  canvas.clear();

  // Copy our texture into the window.
  canvas.copy(
    &square_texture,
    None,
    Rect::new(0, 0, TEXTURE_SIZE, TEXTURE_SIZE),
  )?;

  // Update window's display.
  canvas.present();

  sleep(Duration::new(0, 1_000_000_000u32 / 60));
}
```
