# ggez

- [ggez](#ggez)
  - [Hello world](#hello-world)

## Hello world

```rust
use ggez::event;
use ggez::graphics::{self, Color, Mesh};
use ggez::{Context, GameResult};
use glam::*;

const TEXT_Y: f32 = 0.;
const CIRCLE_Y: f32 = 380.;

struct MainState {
    objects_x: f32,
    circle: Mesh,
    text: graphics::Text,
}

impl MainState {
    fn new(ctx: &mut Context) -> GameResult<MainState> {
        let font = graphics::Font::new(ctx, "/LiberationMono-Regular.ttf")?;
        let text = graphics::Text::new(("Hello world!", font, 48.0));

        // Objects can be instantiated inside draw() if needed, using the mutable Context reference.

        let circle = graphics::Mesh::new_circle(
            ctx,
            graphics::DrawMode::fill(),
            Vec2::new(0.0, 0.0),
            100.0,
            2.0,
            Color::WHITE,
        )?;

        Ok(MainState {
            objects_x: 0.0,
            text,
            circle,
        })
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

        // The coordinates represent:
        //
        // - for the text: top left
        // - for the circle: center
        //
        graphics::draw(ctx, &self.text, (Vec2::new(self.objects_x, TEXT_Y),))?;
        graphics::draw(ctx, &self.circle, (Vec2::new(self.objects_x, CIRCLE_Y),))?;

        // Tell the graphics system to actually put everything on the screen. To be called at the end
        // of draw().
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
