---
title: "Shallow vs Deep Nested Resources in Rails"
date: 2024-08-16T18:39:43+05:30
draft: false
---

In Rails, in order to create routes we've define resources. Resources provide a mapping between an http verb with its URL and a controller action. For eg. to create routes for `posts` we've to add following to our routes.rb:

```ruby
# routes.rb
Rails.application.routes.draw do
  ...

  resources :posts
end
```

This will generate the following seven routes each of it maps to `PostsController` and one of its action
```
HTTP Verb   Path             Controller#Action     Used for
GET         /posts           posts#index          display a list of all posts
GET         /posts/new       posts#new            return an HTML form for creating a new post
POST        /posts           posts#create         create a new post
GET         /posts/:id       posts#show           display a specific post
GET         /posts/:id/edit  posts#edit           return an HTML form for editing a post
PATCH/PUT   /posts/:id       posts#update         update a specific post
DELETE      /posts/:id       posts#destroy        delete a specific post
```

## What are Nested Resources ?

There are cases when we've a resource is logically children another resource. For eg. comments are children of posts, a comment won't exist without a post.

For such cases, nested resources are required. A nested resources can be defined as such:

```ruby
# routes.rb
Rails.application.routes.draw do
  ...

  resources :posts do
    resources :comments
  end
end
```

This will create posts routes as defined earlier and following additional routes:

```
HTTP Verb   Path                               Controller#Action       Used for
GET         /posts/:post_id/comments           comments#index          display a list of all comments
GET         /posts/:post_id/comments/new       comments#new            return an HTML form for creating a new comment
POST        /posts/:post_id/comments           comments#create         create a new comment
GET         /posts/:post_id/comments/:id       comments#show           display a specific comment
GET         /posts/:post_id/comments/:id/edit  comments#edit           return an HTML form for editing a comment
PATCH/PUT   /posts/:post_id/comments/:id       comments#update         update a specific comment
DELETE      /posts/:post_id/comments/:id       comments#destroy        delete a specific comment
```

We can see here that `:post_id` is also available in the route with the comment's `:id`. This can become cumbersome when there is deep nesting:

```ruby
resources :parent_level do
  resources :child_level do
    resources :another_child_level
  end
end

# this will result in routes like
/parent_level/1/child_level/2/another_child_level/3
```

To avoid such scenarios atmost two level deep nesting is allowed. A better way to handle this is to use shallow nesting.

## What's Shallow Nesting ?

To avoid deep nesting as shown previously we must only nest the collection routes of the child. This will be done like this:

```ruby
resources :posts do
  # collection routes of :comments resource
  resources :comments, only: [:index, :show, :new]
end
# member routes of :comments resource
resources :comments, only: [:create, :edit, :update, :delete]
```

This will result in following routes:

```
HTTP Verb   Path                               Controller#Action       Used for
GET         /posts/:post_id/comments           comments#index          display a list of all comments
GET         /posts/:post_id/comments/new       comments#new            return an HTML form for creating a new comment
POST        /posts/:post_id/comments           comments#create         create a new comment
GET         /comments/:id                      comments#show           display a specific comment
GET         /comments/:id/edit                 comments#edit           return an HTML form for editing a comment
PATCH/PUT   /comments/:id                      comments#update         update a specific comment
DELETE      /comments/:id                      comments#destroy        delete a specific comment
```

Here the last four are collection routes which does not contain `:post_id` anymore.

We can also just pass `shallow: true` when defining the `:comments` resource and that'll generate the exact same routes:

```ruby
# routes.rb
Rails.application.routes.draw do
  ...

  resources :posts do
    # Setting shallow: true tells Rails
    # to generate shallow nested routes
    resources :comments, shallow: true
  end
end
```
