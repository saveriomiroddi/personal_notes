# Game programming patterns

- [Game programming patterns](#game-programming-patterns)
  - [Design patterns revisited](#design-patterns-revisited)
    - [2. Command](#2-command)

## Design patterns revisited

### 2. Command

Base form. The actions for each button are hardcoded.

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

With Command class. The actions can be easily swapped.

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

Implement an Actor strategy. Any actor can be easily swapped (and the player can easily control any).

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
