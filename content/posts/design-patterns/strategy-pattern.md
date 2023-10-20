---
title: "Strategy Pattern"
date: 2023-10-14T18:37:05+05:30
draft: false
---

Strategy pattern is a behavioral pattern which is used when there is a need to choose an algorithm on runtime. It's a very good example of open/closed principle.

### What is Open/Closed Principle

> *Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.*

The idea is to add new functionality without changging the existing classes. So how does Strategy Pattern makes sure open/closed principle is used in our code, lets find out.

### Understanding Strategy Pattern with an example

Lets say we're building an AI player for a chess game, we have to define 3 levels for the game - easy, medium and hard. Based on the level the player chooses we need to adjust the difficulty of the AI player.

This means there are 3 algoritms(or strategies) from which the AI player can use to make its next move. We'll call these strategies. There will be one `ChessGame` class which will pick from one of these strategies. So we'll have following classes:

- **HardLevelStrategy** - Defines the algorithm for hard level.
- **MediumLevelStrategy** - Defines the algorithm for medium level.
- **EasyLevelStrategy** - Defines the algorithm for easy level.
- **AiStrategy** - Abstract class to define methods required to be implemented by each strategy.
- **ChessGame** - Uses one of the strategy to make the next move for the AI player.

![Strategy Pattern Chess Game](/design-patterns/strategy-pattern/strategy-pattern-chess-game.png)

```ruby
class AiStrategy

  def next_move
    raise NotImplementedError.new
  end

  def has_move?
    # if ai's king is still present
  end

end

class HardLevelStrategy < AiStrategy

  def next_move
    # Returns next move based on hard level
  end

end

class MediumLevelStrategy < AiStrategy

  def next_move
    # Returns next move based on medium level
  end

end

class EasyLevelStrategy < AiStrategy

  def next_move
    # Returns next move based on easy level
  end

end
```

One of the strategies is picked on runtime based on which level the player chooses. Lets create a `ChessGame` class to use these strategies:

```ruby
class ChessGame

  def initialize(strategy:)
    @strategy = strategy
  end

  def play(player:)
    while true
      if player.has_move?
        player.wait_for_players_move
      else
        return "Player #{player.player_name} lost the game"
      end

      if @strategy.has_move?
        @strategy.next_move
      else
        return "Player #{player.player_name} wins the game"
      end
    end

  end

end

class Player

  attr_accessor :player_name

  def wait_for_players_move
    # wait for player to make the next move
  end

end
```

If the player chooses medium level then we'll pass `MediumLevelStrategy` to the `ChessGame` class to use as the strategy:

```ruby
strategy = MediumLevelStrategy.new
game = ChessGame.new(strategy: strategy)

game.start(player: Player.new)
```

The level strategy of the game is set on the runtime. What about open/closed principle ?

To find out if our design follows the open/closed principle we'll add another level to our game - the random level. To add this new level we'll need to create a new strategy `RandomizedStrategy`:

```ruby
class RandomizedStrategy < AiStrategy

  def next_move
    # Returns next move based on easy level
  end

end
```

As we did before for `MediumLevelStrategy` we can use our new strategy in the `ChessGame` class by passing an isntance into it:

```ruby
strategy = RandomizedStrategy.new
game = ChessGame.new(strategy: strategy)

game.start(player: Player.new)
```

So we created a new class `RandomizedStrategy` and were able to introduce the functionality without doing any modification on our existing classes, which is what the open/closed principle states.

### Wrap Up

In general, the strategy pattern contains the following classes:

- **AbstractStrategy** - An abstract class which defines the methods to be implemented by each strategy.
- **StrategyA** - A strategy class which implements type A strategy.
- **StrategyB** - A strategy class which implements type B strategy.
- **StrategyC** - A strategy class which implements type C strategy.
- **Executor** - Accepts a strategy on runtime and executes the particular algorithm.

![Strategy Pattern Class Diagram](/design-patterns/strategy-pattern/strategy-pattern-class-diagram.png)

There can N number of strategies as long as it implements the methods defined in `AbstractStrategy`.
