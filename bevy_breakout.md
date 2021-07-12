# Bevy breakout

- [Bevy breakout](#bevy-breakout)
  - [General info](#general-info)
  - [Parts](#parts)
  - [Systems tree](#systems-tree)

## General info

- Repo/branch: https://github.com/alice-i-cecile/bevy/tree/better-breakout
- PR: https://github.com/bevyengine/bevy/pull/2094

## Parts

- Resources
  - `Score`: resource

- Components
  - main
    - `Paddle`
      - Collides, Velocity
    - `Ball`
      - Collides, Velocity
    - `Brick`
      - Collides
    - `Scoreboard`: graphics
  - marker
    - `Velocity`
    - `Collides`: used to identify actors that collide

- Systems
  - startup
    - spawn_cameras
    - spawn_paddle
    - spawn_ball
    - spawn_walls
    - spawn_bricks
    - spawn_scoreboard
  - fixed time step (system set)
    - ball_collision
    - paddle_input
    - kinematics
  - standard
    - bound_paddle
    - update_scoreboard
    - exit_on_esc_system

## Systems tree

```
  paddle_input --+
                 |
                 +--> kinematics --> bound_paddle
                 |
ball collision --+
```

There is an additional dependency (paddle_input -> bound_paddle), but it's redundant.
