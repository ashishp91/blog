---
title: "Concerns in Rails"
date: 2023-08-06T17:37:13+05:30
draft: false
tags: ["Ruby"]
---

> Concerns allow us to include modules with methods (both instance and class) and constants into a class so that the including class can use them.

Rails concerns are defined by extending `ActiveSupport::Concern` module. By extending this the module can define two blocks:

- **included**: Any code written inside this block is evaluated in the context of the including class. That means if we define a method in this block then that'll be available to the class in which the concern is included.
- **class_methods**: The methods added inside this block become the class methods on the including class.

Here's is an example where we're creating a Visible module as a concern and using it in Post class:

```ruby
# visible.rb

module Visible
  extend ActiveSupport::Concern

  # The code inside the included block is evaluated
  # in the context of the class that includes the Visible concern.
  # You can write class macros here, and
  # any methods become instance methods of the including class.
  included do
    attr_accessor :visible_to

    def is_visible?
      visible_to.present?
    end
  end

  # The methods added inside the class_methods block (or, ClassMethods module)
  # become the class methods on the including class.
  class_methods do
    def count_all_visible
      all.select { |item| item.is_visible? }
    end
  end

```

and then we use it in `Post` class:

```ruby
class Post
  include Visible

end
```
Post class will have visible_to as an attribute and is_visible? as an instance method while it'll have a class method count_all_visible:

```ruby
post = Post.new
post.visiblt_to = 'reader'
post.is_visible?
# true

Post.count_all_visible
# []
```

Concerns are used to abstract away common functionality. It helps in reusing it in different classes and DRYing out your code.
