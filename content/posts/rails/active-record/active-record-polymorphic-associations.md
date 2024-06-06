---
title: "Polymorphic Associations"
date: 2024-04-12T08:24:33+05:30
draft: false
tags: ["Ruby", "Rails", "ActiveRecord"]
---

### What is Polymorphic Associations ?

With Polymorphic Association a model can belong to more than one models with a single association. Let's take an example, we'll use polymorph associations on the following schema:

![Insert Op](/active-record-polymorphic-associations/polymorph-schema.png)

In the above schema picture belongs to both employee and products. Creating picture model with polymorphic association will look like this:

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

### attr_type column in polymorphic association

If we look at the migration of `Picture` model, we'll see that there is an extra `imageable_type` column:

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

The `imageable_type` column is used to find out which model does the record belong to. In our case if the picture record belongs to products the column will have Product as the value while if the record belongs to employees the column will have Employee value.
