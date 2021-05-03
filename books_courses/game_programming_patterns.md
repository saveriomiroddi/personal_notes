# Game programming patterns

- [Game programming patterns](#game-programming-patterns)
  - [II. Design patterns revisited](#ii-design-patterns-revisited)
    - [2. Command](#2-command)
      - [Undo and Redo](#undo-and-redo)

## II. Design patterns revisited

### 2. Command

A command is a reified method call.

Base form; the actions for each button are hardcoded.

```ruby
class InputHandler
  def handleInput
    case
    when isPressed(BUTTON_X) jump()
    when isPressed(BUTTON_Y) fireGun()
    end
  end
end
```

With Command class; the actions can be easily swapped.

```ruby
class Command
  def execute; raise "abstract"; end
end

class JumpCommand < Command
  def execute
    jump
  end
end

class FireCommand < Command
  def execute
    fireGun
  end
end

class InputHandler
  def initialize
    @buttonX = JumpCommand.new
    @buttonY = FireCommand.new
  end

  def handleInput
    case
    when isPressed(BUTTON_X) @buttonX.execute
    when isPressed(BUTTON_Y) @buttonY.execute
    end
  end
end
```

Implement an Actor strategy; any actor can be easily swapped (and the player can easily control any).

Another application is for the AI - different ones can be swapped, and produce a stream of commands that affect differently the actor passed; this decouples the AI from the actors.
A demo mode can even be implemented with this pattern.

```ruby
class Command
  def execute(actor); raise "abstract"; end
end

class JumpCommand < Command
  def execute(actor)
    actor.jump
  end
end

class InputHandler
  def handleInput
    return \
      case
      when isPressed(BUTTON_X) @buttonX
      when isPressed(BUTTON_Y) @buttonY
      end
  end
end

command = inputHandler.handleInput
command&.execute(actor)
```

#### Undo and Redo

Easily implement undo/redo, by storing the previous state(s).

```ruby
class Command
  def execute; raise "abstract"; end
  def undo; raise "abstract"; end # new
end

# It's easy to implement multiple undos, by storing all the previous states.
#
class MoveUnitCommand <  Command
  def initialize(unit, x, y)
    @unit, @x, @y = unit, x, y
    @xBefore, @yBefore = 0, 0
  end

  def execute
    @xBefore, @yBefore = @unit.x, @unit.y
    @unit.moveTo(@x, @y)
  end

  def undo
    @unit.moveTo(@xBefore, @yBefore)
  end

# Functional approach: return the command, as function(s).
#
def makeMoveUnitCommand(unit, x, y)
  xBefore, yBefore = unit.x, unit.y
  {
    execute: ->() { unit.moveTo(x, y) },
    undo: ->() { unit.moveTo(xBefore, yBefore) },
  }
end
```
