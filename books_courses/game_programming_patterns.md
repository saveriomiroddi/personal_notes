# Game programming patterns

- [Game programming patterns](#game-programming-patterns)
  - [II. Design patterns revisited](#ii-design-patterns-revisited)
    - [2. Command](#2-command)
      - [Undo and Redo](#undo-and-redo)
    - [3. Flyweight](#3-flyweight)
    - [4. Observer](#4-observer)
    - [5. Prototype](#5-prototype)
    - [6. Singleton](#6-singleton)
    - [7. State](#7-state)
  - [III. Sequencing Patterns](#iii-sequencing-patterns)
    - [9. Game loop](#9-game-loop)
    - [10. Update](#10-update)

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

### 3. Flyweight

Simple: store common data into single location (eg. tiles, inside world), and add a reference to the subject class.

If the shared property of an subject instance (eg. is terrain) is queried, one will forward the query to the parent instance (eg. tile).

### 4. Observer

Very well-known and easy structure.

- if the work to do is slow, send it to a thread, but take care of deadlocks
- some complain about memory allocation
  - simple linked list
  - intrusive LL (more space-efficient)
  - double LL (constant-time removal)

Referential problems concepts:

- if observers are just removed, subjects will hold invalid references
  - add a removal call on destruction
- if subjects are just removed, observers will assume they're still observing
  - nice strategy: send a termination message, for observers to unregister
- with GC: lapsed listener problem - if the observers don't deregister, the subjects will hold the references to them

Design problems:

- since observers are dynamically added, it's harder to trace them; if it gets too hard, this pattern may not be well-suited

### 5. Prototype

The chapter is somewhat weak, because it starts with a very poor example (one factory class for each instance type).

Pattern essence: instead of instantiating, implement cloning, and clone existing instances (advantage: state can also be copied, which allows using "template" instances).

### 6. Singleton

Advantages of singleton:

- initialized at runtime; a tree of (lazily initialize) singleton can recursively initialize without needing to be ordered (which would happen if they were statically initialized)
- can be subclasses

Disadvatanges:

- it's essentially a global
  - alternative: get it from something that is already global (e.g. game state/world instance)
  - or use a service locator
- it couples classes (if they are used anywhere)
  - splitting the singleton (e.g. multiple log sytems) requires changes to all the callers
- it's concurrence-unfriendly

### 7. State

Using a finite state machine, we avoid encoding actions as an ugly series of if/else.

```ruby
class PlayerState
  def handleInput(player, input); raise "Abstract"; end
  def update(player); raise "Abstract"; end
end

class StandingState < PlayerState
  def initialize
    @chargeTime = 0
  end

  def enter(player)
    player.setGraphics(IMAGE_STAND)
  end

  def handleInput(player, input)
    if input == PRESS_DOWN
      return new DuckingState
    end

    # Stay in this state.
    #
    nil
  end

  def update(player)
    @chargeTime += 1
    if @chargeTime > MAX_CHARGE
      player.bomb
    end
  end
end

class Player
  def initialize
    @state = nil
  end

  def handleInput(input)
    new_state = @state.handleInput(self, input)

    if new_state
      @state = new_state
      @state.enter(self)
    end
  end

  def update
    @state.update(self)
  end
end
```

For slightly more complex states (e.g. including a weapon), it's possible to store more than one state (e.g. position and weapon); this may require some conditionals for simplicity.

For complex concepts (e.g. AI), this won't work. An option is to use hierarchical state machines, with super/substates; when a state doesn't handle a certain condition, it sends the request up the tree:

```ruby
class OnGroundState < HeroineState
  def handleInput(heroine, input)
    if input == PRESS_B
      # ...
    elsif input == PRESS_DOWN
      # ...
    end
  end
end

class DuckingState < OnGroundState
  def handleInput(heroine, input)
    if input == RELEASE_DOWN
      # ...
    else
      # Didn't handle input, so walk up hierarchy.
      super(heroine, input)
    end
  end
end
```

In order to keep history, a more sophisticated approach is required: the "Pushown automaton" - essentially, a stack of states (used also, for example, for game state handling).

With this, one pushes/pops/replaces states in the stack.

## III. Sequencing Patterns

### 9. Game loop

Variable time step: adjusts to machine speed.

Big problem: it's non-deterministic!

```ruby
lastTime = Time.now

loop do
  current = Time.now
  elapsed = current - lastTime

  processInput()

  update(elapsed)
  render()

  lastTime = current
end
```

Fixed time step, with variable rendering rate: more precise than the previous. Important: the rendering is predictive, therefore not necessarily accurate.

```ruby
previous = Time.now
lag = 0.0

loop do
  current = Time.now
  elapsed = current - previous
  previous = current
  lag += elapsed

  processInput()

  while lag >= MS_PER_UPDATE
    update()
    lag -= MS_PER_UPDATE
  end

  render(lag / MS_PER_UPDATE) # This is the predictive part
end
```

### 10. Update

The concept is simple - encapsulate the update behavior of entities, rather than hardcoding them in the main loop. Some considerations:

- entities modifications order can be critical;
- must handle entities state change (including removal) during update cycle;
- the entities update is strictly related to the architecture, i.e. ECS/deep inheritance.
