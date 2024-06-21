---
title: "Single Table Inheritance"
date: 2024-04-12T09:24:02+05:30
draft: false
tags: ["Ruby", "Rails", "ActiveRecord"]
---

## What is Single Table Inheritance ?

>Single Table Inheritance is a mechanism through which ActiveRecord can share fields and behavior between different models while using a single table.

For eg. let's say we need to create a `Vehicle` model and `Car` model. Since `Vehicle` and `Car` will have common attributes and methods we can use `Single Table Inheritance` on them.
___

### 1. Creating Vehicle and Car model for Single Table Inheritance

* Let's generate vehicles model and a cars model with vehicles as its parent:

```ruby
rails generate model vehicle type:string color:string price:decimal{10.2}
rails generate model car --parent=Vehicle
```

* The `Car` model will look something like this:

```ruby
class Car < Vehicle

end
```
### 2. Understanding relationship between Vehicle and Car

The `Car` class inherits `Vehicle` class which means _all behaviors, associations and public methods from `Vehicle` class are available to `Car` class as well_.

Because `Single Table Inheritance` both the models will share the same table `vehicles` which is the table created for the parent class.

Let's create a `Car` record:

```ruby
Car.create(color: 'Red', price: 10000)

INSERT INTO "vehicles" ("type", "color", "price") VALUES ('Car', 'Red', 10000)
```

We can see that the record is inserted in `vehicles` table even though we're using the `Car` model.

Additionally, _a `type` column is also set in the table with the value Car. This column is used to identify which model does the record belongs to._

Here is how the record is stored in the table:

```sql
select * from vehicles;
 id | type | color |  price  |         created_at         |         updated_at
----+------+-------+---------+----------------------------+----------------------------
  1 | Car  | Red   | 10000.0 | 2024-04-14 09:01:12.180265 | 2024-04-14 09:01:12.180265
(1 row)
```
### 3. Extending the concept to another model

Just like we did above, we can create a `Bus` model and set its parent as the `Vehicle` class. Lets also insert a new record for the `Bus` model in the `vehicles` table:

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

We can see that the new record has `type` set as _Bus_ which means it refers to the `Bus` model.
