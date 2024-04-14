---
title: "Single Table Inheritance"
date: 2024-04-12T09:24:02+05:30
draft: false
tags: ["Ruby", "Rails", "ActiveRecord"]
---

### What is Single Table Inheritance ?

Single Table Inheritance is a mechanism through which ActiveRecord can share fields and behavior between different models. For eg. let's say we need to create a model vehicle and car. Since vehicle and car will share their attributes and methods we can use Single Table Inheritance on them.

Let's generate vehicles model and a cars model with vehicles as its parent:

```ruby
rails generate model vehicle type:string color:string price:decimal{10.2}
rails generate model car --parent=Vehicle
```

The `Car` model will look something like this:

```ruby
class Car < Vehicle
end
```

The `Car` class inherits `Vehicle` in Rails and thus all behaviors, associations and public methods from `Vehicle` are available to `Car` as well. While both the models will share a same table `vehicles` in the database.
Let's create a `Car` record:

```ruby
Car.create(color: 'Red', price: 10000)
```

This executes the following query in the database:
```sql
INSERT INTO "vehicles" ("type", "color", "price") VALUES ('Car', 'Red', 10000)
```

We can see that the record is inserted in `vehicles` table even though we're using the `Car` model. Additionally, a `type` column is also set in the table with the value `Car`. This column is used to identify which model does the record belongs to.

Here is how the record is stored in the table:

```sql
select * from vehicles;
 id | type | color |  price  |         created_at         |         updated_at
----+------+-------+---------+----------------------------+----------------------------
  1 | Car  | Red   | 10000.0 | 2024-04-14 09:01:12.180265 | 2024-04-14 09:01:12.180265
(1 row)
```

Similarly we can create a `Bus` model and insert a new record for `Bus` model in `vehicles` table:

```ruby
rails generate model bus --parent=Vehicle
Bus.create(color: 'Red', price: 10000)
```

Let's see how our table `vehicles` looks like:

```sql
select * from vehicles;
 id | type | color |  price  |         created_at         |         updated_at
----+------+-------+---------+----------------------------+----------------------------
  1 | Car  | Red   | 10000.0 | 2024-04-14 09:01:12.180265 | 2024-04-14 09:01:12.180265
  2 | Bus  | Red   | 10000.0 | 2024-04-14 09:09:52.739512 | 2024-04-14 09:09:52.739512
```

We can see that the new record has `type` set as `Bus` so it refers to the `Bus` model.
