---
title: "Builder Pattern in Ruby"
date: 2023-10-08T12:06:19+05:30
draft: false
tags: ["Ruby", "Rails"]
---

### What is Builder pattern ?

The builder pattern is a creational design pattern used for building complex objects involving several steps to create or require multiple sub objects.

The pattern helps in abstracting away the process of instantiating the sub objects and the relationship among them.

Builder pattern helps in:

- Allowing to create different representation of a complex object.
- Simplifying creating an object with several steps or instantiation of sub objects.

### Understanding with an example

Let's take an example of building a car, in order to build a car we need the following things:

1. Engine
2. Body
3. Wheels

The car will need 1 engine and 1 body while it'll require 4 wheels. The class diagram for this will look like this:

![Car Entities](/design-patterns/builder-pattern/car-entities.png)

Lets code this up:

```ruby
class Car

  def initialize
    @engine = nil
    @wheels = []
    @body = nil
  end

  def set_engine(engine)
    @engine = engine
  end

  def add_wheel(wheel)
    @wheels.push(wheel)
  end

  def set_body(body)
    @body = body
  end

end

class Wheel
  attr_accessor :size
end

class Engine
  attr_accessor :power
end

class Body
  attr_accessor :color
end
```

### Creating Car object without any pattern

Lets start with creating two types of car - a hatchback and an SUV. Since both types of car have different specs we'll need to pass in the specs as we create the individual object.

#### Creating a hatchback

```ruby
hatchback = Car.new

engine = Engine.new
engine.power = 100
hatchback.set_engine(engine)

body = Body.new
body.color = :blue
hatchback.set_body(body)

4.times do
  wheel = Wheel.new
  wheel.size = 15
  hatchback.add_wheel(wheel)
end

return hatchback
```

Hatchback requires a 100 units engine and wheels with size 15 units so we set that while instantiating the objects. Similarly we'll create the SUV car specifying the specs for SUV.

#### Creating an SUV

```ruby
suv = Car.new

engine = Engine.new
engine.power = 150
suv.set_engine(engine)

body = Body.new
body.color = :green
suv.set_body(body)

4.times do
  wheel = Wheel.new
  wheel.size = 20
  suv.add_wheel(wheel)
end

return suv
```

Here we see that we follow the same steps. Now if we had another type of car then we'll need to specify the same steps for the car. This leads to duplicating the same code for every new type of car.

What if we had a new requirement of adding doors object to the car object ? we'll need to make that change for all the types of car. Doesn't look very flexible. Will using builder pattern help here ? Lets see.

### Creating a Car using Builder pattern

In general builder pattern has following classes:

- **AbstractProductBuilder** - defines method signatures to create the Product class's components.
- **Director** - defines the steps for creating the components of the Product object and any relationship among them.
- **ConcreteBuilderA** - defines specific methods for creating a Product A component.
- **ConcreteBuilderB** - defines specific methods for creating a Product B component.

![Builder Pattern](/design-patterns/builder-pattern/builder-pattern-class-diagram.png)

We'll use the same format to create the Car object:

- **CarBuilder** - defines method signature to create the Car class's components.
- **Director** - defines the steps for creating the components of Car object.
- **HatchBackBuilder** - defines specific methods for creating a HatchBack's component.
- **SuvBuilder** - defines specific methods for creating a Suv's component.

![Car using Builder Pattern](/design-patterns/builder-pattern/car-builder-pattern.png)

First we'll define the CarBuilder and Director classes:

```ruby
# Abstract class
class CarBuilder

  def get_wheel
    raise NotImplementedError.new
  end

  def get_engine
    raise NotImplementedError.new
  end

  def get_body
    raise NotImplementedError.new
  end

end

class Director

  def set_builder(builder)
    @builder = builder
  end

  def build_car
    car = Car.new

    engine = @builder.get_engine()
    car.set_engine(engine)

    body = @builder.get_body()
    car.set_body(body)

    4.times do
      wheel = @builder.get_wheel()
      car.add_wheel(wheel)
    end
  end

end
```

The `Director` class encapsulates the steps of creating the `Car` object while the `CarBuilder` class is responsible for providing methods to create the sub objects.

Next lets implement the CarBuilder concrete methods for a hatchback:

```ruby
class HatchBackBuilder < CarBuilder

  def get_wheel
    wheel = Wheel.new
    wheel.size = 15

    wheel
  end

  def get_engine
    engine = Engine.new
    engine.power = 100

    engine
  end

  def get_body
    body = Body.new
    body.color = :blue

    body
  end

end
```

Using the HatchBackBuilder class we can create a hatch back:

```ruby
builder = HatchBackBuilder.new
director = Director.new

director.set_builder(builder)
hatchback = director.build_car

return hatchback
```

Looks much cleaner. Similarly we can create suv using a SuvBuilder class:

```ruby
class SuvBuilder < CarBuilder

  def get_wheel
    wheel = Wheel.new
    wheel.size = 20

    wheel
  end

  def get_engine
    engine = Engine.new
    engine.power = 150

    engine
  end

  def get_body
    body = Body.new
    body.color = :green

    body
  end

end
```

And all we need to change is the builder object passed to the Director:

```ruby
builder = SuvBuilder.new
director = Director.new

director.set_builder(builder)
hatchback = director.build_car

return hatchback
```

### Adding new steps to Car building process

We asked ourselves earlier about what do we need to do to add doors object to the Car object. Using builder pattern encapsulated the steps to create the `Car` in the `Director` class. All we need to do is add the step in that class and it'll be applied to all `Car` types:

```ruby
class Director

  def set_builder(builder)
    @builder = builder
  end

  def build_car
    car = Car.new

    engine = @builder.get_engine()
    car.set_engine(engine)

    body = @builder.get_body()
    car.set_body(body)

    4.times do
      wheel = @builder.get_wheel()
      car.add_wheel(wheel)

      # New step added to car building process
      door = @builder.get_door()
      car.add_door()
    end
  end

end
```
