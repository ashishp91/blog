---
title: "Procs Lambdas and Blocks"
date: 2023-08-06T17:45:58+05:30
draft: false
tags: ["Ruby", "Rails"]
---

## Blocks

Blocks in ruby are anonymous functions that can be passed into methods.

## Calling Blocks

Blocks can be called using the `yield` keyword or by calling `.call`, the latter is called an explicit block while the former is called an implicit block.

- explicit block: Using `.call` to execute a block
```ruby
  def explicit_block(&block)
    block.call
  end
  explicit_block { puts "Explicit block called" }
```
- implicit block: using `yield` to execute a block

```ruby
  # block passed as anonymous function
  def test_method
    yield
  end
  test_method { puts "Block is being run" }

  # block with parameters
  def one_two_three
    yield 1
    yield 2
    yield 3
  end
  one_two_three { |number| puts number * 10 }
  # 10, 20, 30
```
## Proc

a *proc* is an instance of the Proc class and is similar to a block. As opposed to a block, a *proc* is a Ruby object which can be stored in a variable and therefore reused many times throughout a program.

```ruby
# A proc is defined by calling Proc.new followed by a block.
square = Proc.new { |x| x ** 2 }

# Calling a proc with .call method.
square.call(2)
# => 4

# When passing a proc to a method,
# an `&` is used to convert the proc into a block.
[2, 4, 6].collect!(&square)
# => [4, 16, 36]
```

## Lambda

a *lambda* is an object similar to a *proc*. Unlike a *proc*, a *lambda* requires a specific number of arguments passed to it

```ruby
say_something = -> { puts "This is a lambda" }
say_something.call
# => "This is a lambda"

parameterized_lambda = -> (x) { puts x** 2 }
parameterized_lambda.call(2)
# => 4
```

## difference in Procs and Lambdas

- lamdas returns to its calling method and executes the statement after the lamda was called while proc returns from the calling method and doesn't execute the following code.

```ruby
  def proc_demo_method
    proc_demo = Proc.new { return "Only I print!" }
    proc_demo.call
    "But what about me?" # Never reached
  end

  puts proc_demo_method
  # Output
  # Only I print!

  # (Notice that the proc breaks out of the method
  # when it returns the value.)

  def lambda_demo_method
    lambda_demo = lambda { return "Will I print?" }
    lambda_demo.call
    "Sorry - it's me that's printed."
  end

  puts lambda_demo_method
  # Output
  # Sorry - it's me that's printed.

  # (Notice that the lambda returns back to the method
  # in order to complete it.)
```
- Procs do not check the number of arguments, whereas Lambdas do.

```ruby
lam = lambda { |x| puts x } #produces a lambda with one argument
lam.call(2) # prints out 2
lam.call # ArgumentError: insufficient arguments (0 for 1)
lam.call(1,2,3) #ArgumentError: insufficient arguments (3 for 1)

proc = Proc.new { |x| puts x } # creates a proc with one argument
proc.call(2) # prints out 2
proc.call # returns nil
proc.call(1,2,3) # prints 1 and disregards the extra #arguments
```
