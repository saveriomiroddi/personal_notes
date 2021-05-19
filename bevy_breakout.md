# Bevy breakout

- [Bevy breakout](#bevy-breakout)
  - [Parts](#parts)
  - [Systems tree](#systems-tree)

## Parts

- Resources
  - Score
- Startup systems
  - spawn_cameras
  - spawn_paddle
  - spawn_ball
  - spawn_walls
  - spawn_bricks
  - spawn_scoreboard

## Systems tree

                       (C)
  paddle_input +--------------+-> bound_paddle
            (A)|              |(D)
               +-> kinematics +
            (B)|
ball collision +

