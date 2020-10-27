# SDL/Rust
- [SDL/Rust](#sdlrust)
  - [Terminology and general concepts](#terminology-and-general-concepts)
  - [Base operations](#base-operations)
    - [Keys polling consideration](#keys-polling-consideration)
    - [Using textures](#using-textures)
  - [Maximizing the window](#maximizing-the-window)
  - [Audio play](#audio-play)
    - [Callback (pull)](#callback-pull)
    - [Queue (push)](#queue-push)
    - [References](#references)
      - [Wave formulas](#wave-formulas)
      - [Tone frequencies](#tone-frequencies)
  - [Quick template for video drawing](#quick-template-for-video-drawing)

## Terminology and general concepts

In SDL terminology, to "clear" means to fill with color.

## Base operations

Create a window, color it red, and draw in it a red square.

```rust
use sdl2::{event::Event, keyboard::Keycode, pixels::Color, rect::{Point, Rect}, render::Canvas, video::Window, Sdl};
use std::{thread::sleep, time::Duration};

extern crate rand;

fn main() {
  let sdl_context = sdl2::init().unwrap();

  let mut canvas = prepare_canvas(&sdl_context);

  wait_keyboard_event(&sdl_context);

  fill_canvas(&mut canvas);

  let mut event_pump = sdl_context.event_pump().unwrap();

  'main_loop: loop {
    // Polling: all events are caught, not only the current (last) one, e.g. KeyDown->KeyUp.
    // The poll iterator stops once there are no more events in the queue.
    for event in event_pump.poll_iter() {
      match event {
        Event::Quit { .. } |
        Event::KeyDown { keycode: Some(Keycode::Escape), .. } => {
          break 'main_loop;
        }
        _ => println!("Event: {:?}", event),
      }
    }

    draw_filled_rectangle(&mut canvas);

    draw_points(&mut canvas);

    // Update window's display. SDL operations operate on a back buffer, so this is required.
    canvas.present();

    sleep(Duration::new(0, 1_000_000_000 / 2));
  }
}

fn prepare_canvas(sdl_context: &Sdl) -> Canvas<Window> {
  let video_subsystem = sdl_context.video().unwrap();

  // Parameters are: title, width, height
  let window = video_subsystem
    .window("Tetris", 800, 600)
    .position_centered()
    .opengl()
    .build()
    .unwrap();

  window
    .into_canvas() // Convert Window to Canvas (simpler to manipulate)
    .target_texture() // Activate texture rendering support
    .present_vsync() // Enable v-sync
    .build()
    .unwrap()
}

fn wait_keyboard_event(sdl_context: &sdl2::Sdl) {
  let mut event_pump = sdl_context.event_pump().unwrap();

  println!("Press any key...");

  // If we need to wait on the first event (although it may not make sense), we'd first need to
  // pump and flush the queue, since already at program start, there are several events (mainly,
  // Window).
  // See https://wiki.libsdl.org/SDL_FlushEvents.

  /*
  let event_subsystem = sdl_context.event().unwrap();

  event_pump.pump_events();
  event_subsystem.flush_events(0, std::u32::MAX);

  let event = event_pump.wait_event();
  */

  'event_loop: for event in event_pump.wait_iter() {
    if let Event::KeyDown { keycode, .. } = event {
      println!("Keyboard event: {:?}", keycode);
      break 'event_loop;
    } else {
      println!("Other event: {:?}", event);
    }
  }
}

fn fill_canvas(canvas: &mut Canvas<Window>) {
  // Also Color::RGBA is supported.
  //
  canvas.set_draw_color(Color::RGB(255, 0, 0));
  canvas.clear();
}

fn draw_filled_rectangle(canvas: &mut Canvas<Window>) {
  canvas.set_draw_color(Color::RGB(rand::random(), rand::random(), rand::random()));
  canvas.fill_rect(Rect::new(0, 0, 32, 32)).unwrap();
}

fn draw_points(canvas: &mut Canvas<Window>) {
  canvas.set_draw_color(Color::RGB(rand::random(), rand::random(), rand::random()));

  let points = (10..100)
    .into_iter()
    .map(|x| Point::new(x, 10))
    .collect::<Vec<Point>>();

  // Watch out! Very ugly signature. Takes a slice.
  canvas.draw_points(&points[..]).unwrap();

  canvas.draw_point(Point::new(13, 10)).unwrap();
}
```

### Keys polling consideration

Watch out! In the context of a regular game, keys handling is synchronous; if input is detected, action is taken immediately.

In emulators, keys handling is asynchronous: if input is detected, there could be several cycles before the input is detected, so resetting the keys on each cycle won't work, since the event for the given event will be fired at relatively large intervals, causing cycle in between to miss it.

In this context, handling keys in terms of Down/Up is probably the appropriate solution.

### Using textures

Textures are the preferred way to draw in SDL, since they're faster, however, for per-pixel drawing, `draw_point()`/`draw_points()` are faster.

```rust
// Create and fill the texture, outside the main loop.
let texture_creator = canvas.texture_creator();
let mut texture = texture_creator.create_texture_target(None, TEXTURE_SIZE, TEXTURE_SIZE)?;

canvas.with_texture_canvas(&mut texture, |texture| {
  texture.set_draw_color(Color::RGB(0, 255, 0));
  texture.clear();
})?;

// Copy the texture.
canvas.copy(
  &texture,
  None,
  Rect::new(0, 0, TEXTURE_SIZE, TEXTURE_SIZE),
)?;
```

Note that the

## Maximizing the window

```rust
// Set it as maximized (ok also is in the builder).
//
let mut window = window.maximized();

// Then wait for the event, which gives the data. Note that this logic consumes the events (and
// generates a new one).
//
let mut event_pump = sdl_context.event_pump().unwrap();

for event in event_pump.poll_iter() {
  if let Event::Window { win_event: WindowEvent::SizeChanged(new_width, new_height), .. } = event {
    window.set_size(new_width as u32, new_height as u32).unwrap();
    break;
  }
}
```

## Audio play

There are two ways of playing audio:

- callback: a struct provides a trait method that is called by the audio subsystem at regular intervals, providing a mapped array for writing the samples;
- push: an array with the samples is sent to the audio subsystem.

Besides structure, the main difference is that callback is invoked indefinitely (until paused/dropped), while when pushing, the samples are finite.

Common code:

```rust
use sdl2::{audio::AudioSpecDesired, AudioSubsystem};
use std::f64::consts::PI;
use std::time::Duration;
use std::thread;

const AMPLITUDE: i16 = i16::MAX / 16; // Volume (i16::MAX = max)
const DEVICE_FREQUENCY: u32 = 44100; // in Herz

let sdl_context = sdl2::init().unwrap();
let audio_subsystem = sdl_context.audio().unwrap();
let desired_spec = AudioSpecDesired {
    freq: Some(DEVICE_FREQUENCY as i32),
    channels: Some(1),
    samples: None,
};
```

### Callback (pull)

```rust
use sdl2::audio::AudioCallback;

struct SimpleCallback {
    sample_i: u32,
    tone_frequency: f64,
}

fn prepare_callback(
    audio_subsystem: &AudioSubsystem,
    desired_spec: &AudioSpecDesired,
    tone_frequency: f64,
) -> sdl2::audio::AudioDevice<SimpleCallback> {
    impl AudioCallback for SimpleCallback {
        type Channel = i16;

        fn callback(&mut self, out: &mut [i16]) {
            let period = DEVICE_FREQUENCY as f64 / self.tone_frequency;

            for input in out.iter_mut() {
                let period_number = self.sample_i as f64 / period;
                let scale_factor = (period_number * 2.0 * PI).sin();
                let sample = (AMPLITUDE as f64 * scale_factor) as i16;

                *input = sample;

                self.sample_i += 1;
            }
        }
    }

    let audio_device = audio_subsystem
        .open_playback(None, desired_spec, |_spec| SimpleCallback {
            sample_i: 0,
            tone_frequency: tone_frequency,
        })
        .unwrap();

    audio_device.resume();

    audio_device
}

let audio_device = prepare_callback(&audio_subsystem, &desired_spec, 750.0);

// The queue will continue until it's closed (/dropped).
//
thread::sleep(Duration::from_millis(400));
audio_device.close_and_get_callback();
```

### Queue (push)

```rust
fn prepare_queue(
    audio_subsystem: &AudioSubsystem,
    desired_spec: &AudioSpecDesired,
    tone_frequency: f64,
    duration: u64,
) -> sdl2::audio::AudioQueue<i16> {
    let period = DEVICE_FREQUENCY as f64 / tone_frequency;
    let wave_samples_count = DEVICE_FREQUENCY * duration as u32 / 1000; // approx.

    let wave_samples = (0..wave_samples_count)
        .into_iter()
        .map(|sample_i| {
            let period_number = sample_i as f64 / period;
            let scale_sign = (period_number * 2.0 * PI).sin();
            (AMPLITUDE as f64 * scale_sign) as i16
        })
        .collect::<Vec<i16>>();

    let audio_queue = audio_subsystem
        .open_queue::<i16, _>(None, &desired_spec)
        .unwrap();

    audio_queue.queue(&wave_samples);

    audio_queue.resume();

    audio_queue
}

let audio_queue = prepare_queue(&audio_subsystem, &desired_spec, 750.0, 400);

// In this case, the queue will stop as soon as the samples are finished.
//
std::thread::sleep(Duration::from_millis(1200));
```

### References

#### Wave formulas

```rust
// Sine wave; source: https://stackoverflow.com/a/10111570
//
let period_number = self.sample_i as f64 / period;
let scale_factor = (period_number * 2.0 * PI).sin();
let sample = (AMPLITUDE as f64 * scale_factor) as i16;

// Square wave (via modulus)
//
let half_period_number = sample_i / (period as u32 / 2);
let scale_sign = if half_period_number % 2 == 0 { 1 } else { -1 };
let sample_value = (AMPLITUDE * scale_sign) as i16;

// Square wave (sign of a sine wave)
//
let period_number = sample_i as f64 / period;
let scale_sign = (period_number * 2.0 * PI).sin().signum(); // difference: signum()
let sample = (AMPLITUDE as f64 * scale_sign) as i16;
```

#### Tone frequencies

- D: 293.665
- E: 329.628
- F: 349.228
- G: 391.995
- A: 440.000
- B: 493.883
- c: 554.365
- d: 587.330
- PC_SPEAKER: 750.0

## Quick template for video drawing

Interface and Color struct:

```rust
pub struct Color {
  pub r: u8,
  pub g: u8,
  pub b: u8,
}

// Interface for drawing to a canvas, and handling events; intentionally designed to be as simple a
// possible.
//
// Don't try pixel reads, it's been a complete trainwreck.
//
pub trait Interface {
  // Logical width (resolution as seen by the program).
  // Underlying libraries may use different data types, but u16 is the one that makes sense.
  //
  const CANVAS_WIDTH: u16 = 1024;
  const CANVAS_HEIGHT: u16 = 768;

  // Initializes the canvas, and maximizes the window.
  //
  fn init(window_title: &str) -> Self;

  // Does not update the canvas; must invoke update_canvas().
  //
  fn write_pixel(&mut self, x: u16, y: u16, color: Color);

  fn update_canvas(&mut self);

  // Wait for keypress; if the key sent if a quit combination (ie. Alt+F4), the program will exit.
  //
  fn wait_keypress(&mut self);
}
```

Implementing type:

```rust
use crate::interface::Interface;

use sdl2::{event::{Event, WindowEvent}, pixels, rect::Point, render, video::Window, EventPump};

const TOP_BORDER_START_SIZE: u16 = 0;
const LEFT_BORDER_START_SIZE: u16 = 0;

pub struct Sdl2Interface {
  event_pump: EventPump,
  canvas: render::Canvas<Window>,

  // Compensate for proportions not equal to the screen.
  //
  top_border_size: i32,
  left_border_size: i32,
}

impl Interface for Sdl2Interface {
  fn init(window_title: &str) -> Self {
    let sdl_context = sdl2::init().unwrap();

    // The resizing (due to `maximized()`) is handled below, by process_events().
    //
    let window = sdl_context
      .video()
      .unwrap()
      .window(
        window_title,
        Self::CANVAS_WIDTH as u32,
        Self::CANVAS_HEIGHT as u32,
      )
      .maximized()
      .position_centered()
      .resizable()
      .build()
      .unwrap();

    let event_pump = sdl_context.event_pump().unwrap();

    let canvas = window.into_canvas().present_vsync().build().unwrap();

    let mut sdl2_canvas = Sdl2Interface {
      event_pump,
      canvas,
      top_border_size: TOP_BORDER_START_SIZE as i32,
      left_border_size: LEFT_BORDER_START_SIZE as i32,
    };

    sdl2_canvas.process_events(false);

    sdl2_canvas
  }

  fn write_pixel(&mut self, x: u16, y: u16, color: Color) {
    if x < Self::CANVAS_WIDTH && y < Self::CANVAS_HEIGHT {
      let pixel = pixels::Color::RGB(color.r, color.g, color.b);

      self.canvas.set_draw_color(pixel);

      self.canvas
        .draw_point(Point::new(
          self.left_border_size + x as i32,
          self.top_border_size + y as i32,
        ))
        .unwrap();
    }
  }

  fn update_canvas(&mut self) {
    self.canvas.present();
  }

  fn wait_keypress(&mut self) {
    self.process_events(true);
  }
}

impl Sdl2Interface {
  fn process_events(&mut self, blocking: bool) {
    loop {
      let event = if blocking {
        Some(self.event_pump.wait_event())
      } else {
        self.event_pump.poll_event()
      };

      match event {
        Some(Event::KeyDown { .. }) => break,
        Some(Event::KeyUp { .. }) => break,
        Some(Event::Window {
          win_event: WindowEvent::SizeChanged(new_width, new_height),
          ..
        }) => {
          self.update_window_dimensions(new_width, new_height);
        }
        Some(Event::Quit { .. }) => std::process::exit(0),
        // This happens only for non-blocking events.
        //
        None => break,
        // Ignore all the other events.
        //
        _ => {}
      }
    }
  }

  // Reacts to window resizing events; takes care of clearing, centering, and updating the scale.
  //
  fn update_window_dimensions(&mut self, window_width: i32, window_height: i32) {
    let min_scale = f32::min(
      (window_width as f32) / (Self::CANVAS_WIDTH as f32).floor(),
      (window_height as f32) / (Self::CANVAS_HEIGHT as f32).floor(),
    );

    self.canvas.set_scale(min_scale, min_scale).unwrap();

    // The FP accuracy is not worth considering.
    //
    self.top_border_size = ((window_height as f32 / min_scale) as i32 - Self::CANVAS_HEIGHT as i32) / 2;

    self.left_border_size = ((window_width as f32 / min_scale) as i32 - Self::CANVAS_WIDTH as i32) / 2;

    // If we don't clear, if a part of the canvas is not covered due to mismatch between the
    // screen and the window, will have undefined content.
    //
    self.canvas.set_draw_color(pixels::Color::RGB(0, 0, 0));
    self.canvas.clear();
  }
}
```
