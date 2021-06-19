# Macroquad

- [Macroquad](#macroquad)
  - [Hello world](#hello-world)
  - [Draw](#draw)
    - [Text](#text)
    - [Images/Sprites](#imagessprites)
    - [Tiles](#tiles)
  - [Resources handling/ECS](#resources-handlingecs)
  - [Audio/Sound](#audiosound)
  - [Physics](#physics)
      - [Manually checking sprite <> tile bottom collision](#manually-checking-sprite--tile-bottom-collision)
  - [Input](#input)
  - [Scene management/ECS](#scene-managementecs)
  - [Misc](#misc)
    - [I/O](#io)
    - [Random](#random)

## Hello world

```rs
// All the functions/constants are in the prelude.
//
use macroquad::prelude::*;

fn window_conf() -> Conf {
    // Defaults
    Conf {
        window_title: "".into(),
        window_width: 800,
        window_height: 600,
        window_resizable: true,
        sample_count: 1,          // MSAA
        high_dpi: false,
        ..Default::default()
    }
}

#[macroquad::main("BasicShapes")]
async fn main() -> Result<(), Box<dyn error::Error>> {
    loop {
        // blah...
        next_frame().await
    }
}
```

## Draw

```rs
clear_background(RED);

draw_line(40.0, 40.0, 100.0, 200.0, 15.0, BLUE);
draw_rectangle(screen_width() / 2.0 - 60.0, 100.0, 120.0, 60.0, GREEN);
draw_circle(screen_width() - 30.0, screen_height() - 30.0, 15.0, YELLOW);
```

### Text

```rs
draw_text("IT WORKS!", 20.0, 20.0, 30.0, DARKGRAY);
```

### Images/Sprites

See the [resources handling](#resources-handlingecs) section for storage/load.

```rs
let sprite: Texture2D = load_texture("assets/Whale/Whale(76x66)(Orange).png").await?;

// The `draw_texture()` function has no texture params.
draw_texture_ex(
    sprite,                                          // texture
    0., 0.,                                          // x, y
    WHITE,                                           // color
    DrawTextureParams {                              // params
        source: Some(Rect::new(0.0, 0.0, 76., 66.)), // source (eg. from a sprite sheet)
        ..Default::default()
    },
);
```

### Tiles

The project includes the `macroquad-tiled` crate, which supports the [Tiled](https://www.mapeditor.org) map editor.

```rs
const LAYER: &str = "main layer"; // Name in `assets/map.json`

// load map and textures [files] here ...

let tiled_map: Map = macroquad_tiled::load_map(
    &tiled_map_json,                                                // data
    &[("tileset.png", tileset), ("decorations1.png", decorations)], // textures
    &[],                                                            // external_tilesets
).unwrap(); // not compatible with `Box<dyn Error>`

tiled_map.draw_tiles(
    LAYER,
    Rect::new(0.0, 0.0, screen_width(), screen_height()), // dest
    None,                                                 // source
);

let tile: &Option<Tile> = tiled_map.get_tile(LAYER, x_u32, y_u32);

let tiles: TilesIterator = tiled_map.tiles();
```

## Resources handling/ECS

Resources are typically stored and loaded in the global storage; this is also because they're async, so they can't be loaded in a standard initializer or so.

```rs
pub struct Resources {
    pub title_texture: Texture2D,
}

impl Resources {
    pub async fn new() -> Result<Resources, Box<dyn error::Error>> {
        Ok(Resources { title_texture: load_texture("title.png").await? })
    }
}

async fn main() -> Result<(), Box<dyn error::Error>> {
    let resources_loading = start_coroutine(async move {
        let resources = Resources::new().await.unwrap();
        storage::store(resources);
    });

    while !resources_loading.is_done() {
        clear_background(BLACK);
        let text = format!("Loading resources {}", ".".repeat(((get_time() * 2.) as usize) % 4));
        draw_text(text, screen_width() / 2. - 160., screen_height() / 2., 40., WHITE);
        next_frame().await;
    }
}

// In order to have a consistent assets/resource location across O/Ss, use:
set_pc_assets_folder("assets");

let resources = storage::get::<Resources>(); // get_mut() also availale
let title_texture = resources.title_texture;
```

## Audio/Sound

Formats supported out of the box: `wav`, `ogg`.

```rust
let sound1 = audio::load_sound("sound.wav").await?;
audio::play_sound_once(sound1);

audio::set_sound_volume(sound, volume_f32);
audio::play_sound(sound, PlaySoundParams { looped: true, volume: 0.3});
audio::stop_sound(sound);
```

## Physics

The project includes the `macroquad-platformer` crate.

```rs
const JUMP_SPEED: f32 = -700.0;
const GRAVITY: f32 = 2000.0;
const MOVE_SPEED: f32 = 300.0;

// The actor position is owned/managed by the world (see move_*() calls in the game loop).
struct Player {
    collider: Actor,
    speed: Vec2,
}

// 1-dimensional array of tile present/absent.
let static_colliders = tiled_map
    .tiles("main layer", None)
    .map(|(_x, _y, tile)| tile.is_some())
    .collect();

// Create the world, with the collision map!
let mut world = World::new();

world.add_static_tiled_layer(
    static_colliders,
    tile_width, tile_height,
    map_width,
    1,                       // layer tag
);

// Use the collsion box dimensions here (which can be different from the sprite ones)
let collider: Actor = world.add_actor(vec2(200.0, 100.0), 36, 66);

let mut player = Player { collider, speed: vec2(0., 0.) };

// GAME LOOP

// Get the player (Actor) position; it's owned by the world; so the collider is just used as identifier.
let player_pos: Vec2 = world.actor_pos(player.collider);
let on_ground: bool = world.collide_check(player.collider, player_pos + vec2(0., 1.));

if on_ground == false {
    player.speed.y += GRAVITY * get_frame_time();
}

if is_key_down(KeyCode::Right) {
    player.speed.x = MOVE_SPEED;
} else if is_key_down(KeyCode::Left) {
    player.speed.x = -MOVE_SPEED;
} else {
    player.speed.x = 0.;
}

if is_key_pressed(KeyCode::Space) {
    if on_ground {
        player.speed.y = JUMP_SPEED;
    }
}

// Same considerations of the player position - for that reason, `player.collider` is not mutably passed.
world.move_h(player.collider, player.speed.x * get_frame_time());
world.move_v(player.collider, player.speed.y * get_frame_time());
```

#### Manually checking sprite <> tile bottom collision

Just for reference:

```rs
let sprite_bottom_point = vec2(sprite_pos.x + sprite_width / 2., sprite_pos.y + sprite_height);

// The Map type has higher level functions; tiled::Map has the properties, including:
// map width/height, tile width/height
let raw_tiled_map: tiled::Map = tiled_map.raw_tiled_map;

let tile_below = vec2(
    sprite_bottom_point.x / screen_width * raw_tiled_map.width as f32,
    sprite_bottom_point.y / screen_height * raw_tiled_map.height as f32,
);

if let None = tiled_map.get_tile(LAYER, tile_below.x as u32, tile_below.y as u32) { /* fall */ }
```

## Input

```rs
if is_key_down(KeyCode::Right) { /* ... */ }
```

## Scene management/ECS

```rs
struct Resources {
    whale: Texture2D,
    physics: World,
}

let mut resources: impl DerefMut<Target = Resources> = collections::storage::get_mut::<Resources>();
let collider: Actor = resources.physics.add_actor(vec2(200.0, 100.0), 36, 66),

impl Node for Player {
    fn draw(node: RefMut<Self>) {
        let resources = storage::get_mut::<Resources>();

        let pos = resources.physics.actor_pos(node.collider);

        draw_texture_ex(
            resources.whale,
            pos.x - 20.,
            pos.y,
            WHITE,
            DrawTextureParams {
                source: Some(Rect::new(0.0, 0.0, 76., 66.)),
                ..Default::default()
            },
        );
    }

    fn update(mut node: RefMut<Self>) {
        let world = &mut storage::get_mut::<Resources>().physics;

        let pos = world.actor_pos(node.collider);
        let on_ground = world.collide_check(node.collider, pos + vec2(0., 1.));

        if on_ground == false {
            node.speed.y += Self::GRAVITY * get_frame_time();
        }

        if is_key_down(KeyCode::Right) {
            node.speed.x = Self::MOVE_SPEED;
        } else if is_key_down(KeyCode::Left) {
            node.speed.x = -Self::MOVE_SPEED;
        } else {
            node.speed.x = 0.;
        }

        if is_key_pressed(KeyCode::Space) {
            if on_ground {
                node.speed.y = Self::JUMP_SPEED;
            }
        }

        world.move_h(node.collider, node.speed.x * get_frame_time());
        world.move_v(node.collider, node.speed.y * get_frame_time());
    }
}
```

## Misc

### I/O

```rs
let file = load_file("path/to/file").await?
// Lossily load a string
let tiled_map_json = load_string("assets/map.json").await?;
```

### Random

```rs
// Shuffle an array!
//
use macroquad::rand::ChooseRandom;
my_vec.shuffle();
```
