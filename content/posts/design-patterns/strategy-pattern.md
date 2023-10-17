---
title: "Strategy Pattern"
date: 2023-10-14T18:37:05+05:30
draft: true
---

Strategy pattern is a behavioral pattern which is used when there is a need to choose an algorithm on runtime based on some conditions.

For example let's say we're building an ai player for a chess game, we define 3 levels for the game - easy, medium and hard. Based on the level the player chooses the ai needs to make the next move. We create 3 strategies to encapsulate the levels:

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

We use these strategies to predict the next move. Lets create a chess game class to use these strategies:

```ruby
class ChessGame

  def initialize(strategy:)
    @strategy = strategy
  end

  def play(player:)

    while true
      if player.has_move?
        player.next_move
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

end
```

The player chooses the level of the game on run time and based on that appropriate strategy is set in the ChessGame class. Lets say player chooses medium level then:

```ruby
strategy = MediumLevelStrategy.new
game = ChessGame.new(strategy: strategy)

game.start(player: Player.new)
```

The level strategy of the game is set on the runtime. We'll also notice that this follows the open/closed principle which states that:

> Software entities, like classes, modules, objects etc, must be open for extension but closed for modification.

To test if our design adheres to the open/closed principle lets say we need to add another level - randomized level. In order to achieve this we'll need to create a new strategy `RandomizedStrategy`:

```ruby
class RandomizedStrategy < AiStrategy

  def next_move
    # Returns next move based on easy level
  end

end
```

We can use this new strategy in our ChessGame class:

```ruby
strategy = RandomizedStrategy.new
game = ChessGame.new(strategy: strategy)

game.start(player: Player.new)
```

As we can see, we created a new class `RandomizedStrategy` to use our new strategy. We were able to introduce a new functionality without doing any modification on our existing classes, hence following the open/closed principle.
