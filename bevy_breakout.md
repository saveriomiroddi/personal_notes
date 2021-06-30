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
  - Score
- Startup systems
  - spawn_cameras
  - spawn_paddle
  - spawn_ball
  - spawn_walls
  - spawn_bricks
  - spawn_scoreboard

## Systems tree

```
                      (->B)
  paddle_input --+-----------------+--> bound_paddle
            (->C)|                 |(D->)
                 +--> kinematics --+
            (->A)|
ball collision --+
```
