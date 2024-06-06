---
title: "Setting Up Devise With Rails 7"
date: 2024-06-06T10:24:29+05:30
draft: false
---

In this blog we're going to look at how to setup `devise` gem with `rails 7` using `importmaps`. If you want to just setup `devise` without knowing the nitty gritty follow `tldr;` otherwise scroll past it for the full blog.

---

## tldr;

Follow thsese steps to setup `devise` with `rails`. You must have `rails 7.x` installed on your system.

- Set up new app
```bash
rails new devise-tutorial
```
- Inside the `devise-tutorial` folder create a new controller
```bash
rails g controller Home index
```
- Add `devise` gem to your Gemfile
```bash
bundle add devise
```
- Install `devise` configuration into your Rails app
```bash
bundle exec rails g devise:install
```
- Generate `devise views`
```bash
rails g devise:views
```
- Generate a user model for `devise`
```bash
bundle exec rails g devise user
```
- Apply rails migration
```bash
bundle exec rails db:migrate
```
- Restart the rails server if already running or start the rails app using `rail s`
- Go to `http://localhost:3000/users/sign_in`

## THE END

---
## What is devise ?

Devise is a gem which is used in Rails projects to add authentication to applications. It covers complete functionality like ***login, signup, reset password, user tracking, user account locking and more.***

Using devise enables us to not have to implement all the components required authentication from scratch. Additionally devise also supports omniauth.

## Setting up devise with Rails 7

1. Let's create a new Rails 7 app. I am working with rails 7.1.3.4 and ruby version 3.2.3:
```bash
➜  devise-tutorial git:(main) ✗ rails -v
Rails 7.1.3.4
➜  devise-tutorial git:(main) ✗ ruby -v
ruby 3.2.3 (2024-01-18 revision 52bb2ac0a6) [x86_64-linux]
```
- To create a new app run the following command:
```bash
rails new devise-tutorial
```

- We're going to generate a new controller. In the devise-tutorial folder, run `rails g controller Home index`. This will generate a home_controller.

```bash
➜  devise-tutorial git:(main) ✗ rails g controller Home index
      create  app/controllers/home_controller.rb
       route  get 'home/index'
      invoke  erb
      create    app/views/home
      create    app/views/home/index.html.erb
      invoke  test_unit
      create    test/controllers/home_controller_test.rb
      invoke  helper
      create    app/helpers/home_helper.rb
      invoke    test_unit
```
  - Our `HomeController` should look like this:
```ruby
class HomeController < ApplicationController
  def index
  end
end
```
- Let's change `app/views/home/index.html.erb`, I am going to change it to have an `h2`, you can change however you want
```html
<h2>Hello World</h2>
```
- In our `routes.rb` file we should have `get home/index` added, let's make it the **root path**. The `routes.rb` should look like this:

```ruby
Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Reveal health status on /up that returns 200 if the app boots with no exceptions, otherwise 500.
  # Can be used by load balancers and uptime monitors to verify that the app is live.
  get "up" => "rails/health#show", as: :rails_health_check

  # Defines the root path route ("/")
  # root "posts#index"
  root to: "home#index"
end
```
- Run the rails server and check it lands on the home controller
![Landing Page](/devise/setting-up-devise-with-rails-7/landing-page.png)

2. Add the `devise` gem using `bundle add devise`
```bash
➜  devise-tutorial git:(main) ✗ bundle add devise
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Fetching devise 4.9.4
Installing devise 4.9.4
```
Your Gemfile should have an additional line specifying devise now
![Landing Page](/devise/setting-up-devise-with-rails-7/devise-in-gemfile.png)

3. Configure `devise` by running `bundle exec rails g devise:install`
```bash
➜  devise-tutorial git:(main) ✗ bundle exec rails g devise:install
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Depending on your application's configuration some manual setup may be required:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

     * Required for all applications. *

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

     * Not required for API-only Applications *

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

     * Not required for API-only Applications *

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

     * Not required *

===============================================================================
```
This will create devise configuration initializer file `config/initializers/devise.rb` which will load devise when rails server starts up.
- You can additionally do the configure the mailer to `localhost` in development environment so you can see mails in the rails logs.
```ruby
# Add in config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
- In order to enable flash notifictions from devise you can add following html in the application.html.erb
```html
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```
Your application.html.erb will look something like this
```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>DeviseTutorial</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%# DEVISE FLASH NOTIFICATION html GOES HERE %>
    <p class="notice"><%= notice %></p>
    <p class="alert"><%= alert %></p>

    <%= yield %>
  </body>
</html>
```
4. Generate `devise` views by running `rails g devise:views`
```bash
➜  devise-tutorial git:(main) ✗ rails g devise:views
      invoke  Devise::Generators::SharedViewsGenerator
      create    app/views/devise/shared
      create    app/views/devise/shared/_error_messages.html.erb
      create    app/views/devise/shared/_links.html.erb
      invoke  form_for
      create    app/views/devise/confirmations
      create    app/views/devise/confirmations/new.html.erb
      create    app/views/devise/passwords
      create    app/views/devise/passwords/edit.html.erb
      create    app/views/devise/passwords/new.html.erb
      create    app/views/devise/registrations
      create    app/views/devise/registrations/edit.html.erb
      create    app/views/devise/registrations/new.html.erb
      create    app/views/devise/sessions
      create    app/views/devise/sessions/new.html.erb
      create    app/views/devise/unlocks
      create    app/views/devise/unlocks/new.html.erb
      invoke  erb
      create    app/views/devise/mailer
      create    app/views/devise/mailer/confirmation_instructions.html.erb
      create    app/views/devise/mailer/email_changed.html.erb
      create    app/views/devise/mailer/password_change.html.erb
      create    app/views/devise/mailer/reset_password_instructions.html.erb
      create    app/views/devise/mailer/unlock_instructions.html.erb

```
These views are used by devise controllers. For eg `devise/sessions/new.html.erb` is used to render the sign_in page rendered by the devise's `SessionsController`.

5. Generate the user model for devise

```bash
➜  devise-tutorial git:(main) ✗ bundle exec rails g devise user
      invoke  active_record
      create    db/migrate/20240606060617_devise_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      insert    app/models/user.rb
       route  devise_for :users
```
These models are used to store the user info who have signed up into your application.
- The user model will look like this
```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```
The special keywords are the modules which devise provides out of the box. When a module is enabled it requires to persist some data relevant to the module in the database, and hence it's required to add it in the User model.

Similarly the user migration also have similar keywords
```ruby
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.string   :current_sign_in_ip
      # t.string   :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```
These also needs to be commented out when the module is enabled.
- Run `rails db:migrate` to apply the changes onto the database
```ruby
➜  devise-tutorial git:(main) ✗ rails db:migrate
== 20240606060617 DeviseCreateUsers: migrating ================================
-- create_table(:users)
   -> 0.0016s
-- add_index(:users, :email, {:unique=>true})
   -> 0.0005s
-- add_index(:users, :reset_password_token, {:unique=>true})
   -> 0.0004s
== 20240606060617 DeviseCreateUsers: migrated (0.0026s) =======================
```
- Restart the Rails server.

---

## Test if devise works

- Let's first see the routes added by devise
```
➜  devise-tutorial git:(main) ✗ rails routes | grep /users/
                        new_user_session GET    /users/sign_in(.:format)                                                                          devise/sessions#new
                            user_session POST   /users/sign_in(.:format)                                                                          devise/sessions#create
                    destroy_user_session DELETE /users/sign_out(.:format)                                                                         devise/sessions#destroy
                       new_user_password GET    /users/password/new(.:format)                                                                     devise/passwords#new
                      edit_user_password GET    /users/password/edit(.:format)                                                                    devise/passwords#edit
                           user_password PATCH  /users/password(.:format)                                                                         devise/passwords#update
                                         PUT    /users/password(.:format)                                                                         devise/passwords#update
                                         POST   /users/password(.:format)                                                                         devise/passwords#create
                cancel_user_registration GET    /users/cancel(.:format)                                                                           devise/registrations#cancel
                   new_user_registration GET    /users/sign_up(.:format)                                                                          devise/registrations#new
                  edit_user_registration GET    /users/edit(.:format)                                                                             devise/registrations#edit

```
By default devise enables sign_in, sign_up, user edit and password reset as can be seen in the routes.
- Go to one of these particular route to check if devise works. Visit `localhost:3000/users/sign_in` and you should see something like this:

![Devise Login](/devise/setting-up-devise-with-rails-7/devise-login.png)
