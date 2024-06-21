---
title: "Polymorphic Associations"
date: 2024-04-12T08:24:33+05:30
draft: false
tags: ["Ruby", "Rails", "ActiveRecord"]
---

### What is Polymorphic Associations ?

> With Polymorphic Association a model can belong to more than one models with a single association.

Let's take an example, we'll use polymorphic associations as shown in the following schema:

![Insert Op](/active-record-polymorphic-associations/polymorph-schema.png)

In the above schema `pictures` belongs_to both `employees` and `products`. Adding the polymorphic association in the `Picture` model will look like this:

```ruby
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end

class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end

class Picture < ApplicationRecord
  belongs_to :product, as: :imageable
  belongs_to :employee, as: :imageable
end
```

Now we can use `Picture` model with `Product` and `Employee`:

```ruby
product = Product.first
product.pictures

employee = Employee.first
employee.pictures
```
___
### How does Picture model identify if a record belongs to Product or Employee ?

Looking at the migration of `Picture` model, we can see that there is an extra `imageable_type` column:

```ruby
class CreatePictures < ActiveRecord::Migration[7.1]
  def change
    create_table :pictures do |t|
      t.string  :name
      t.bigint  :imageable_id

      # imageable_type refers to the model to which this record belongs to
      t.string  :imageable_type

      t.timestamps
    end

    add_index :pictures, [:imageable_type, :imageable_id]
  end
end
```

> The `imageable_type` column is used to find out which model does the record belong to. All tables with polymorphic associations will have this `attr_type` column.

In our case if the `pictures` record belongs to `products` the `imageable_type` column will have value _Product_ while if the record belongs to `employees` the column will have _Employee_ value.
