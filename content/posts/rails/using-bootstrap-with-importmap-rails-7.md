---
title: "Using Bootstrap With Importmap Rails 7"
date: 2024-06-06T12:09:06+05:30
draft: true
---
## Setting up bootstrap js

➜  devise-tutorial git:(main) ✗ bin/importmap pin bootstrap
Pinning "bootstrap" to vendor/javascript/bootstrap.js via download from https://ga.jspm.io/npm:bootstrap@5.3.3/dist/js/bootstrap.esm.js
Pinning "@popperjs/core" to vendor/javascript/@popperjs/core.js via download from https://ga.jspm.io/npm:@popperjs/core@2.11.8/lib/index.js

### Add line in application.js
import "popper"
import "bootstrap"

### Add in config/initializers/assets.rb

Rails.application.config.assets.precompile += %w(bootstrap.min.js popper.js)



## Setting up bootstrap css

➜  devise-tutorial git:(main) ✗ mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss


add bootstrap gem

bundle add bootstrap
bundle add sassc-rails

After running the command you should have following lines in your Gemfile:

gem "bootstrap", "~> 5.3"
gem "sassc-rails", "~> 2.1"

➜  devise-tutorial git:(main) ✗ bundle install
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Fetching bootstrap 5.3.3
Installing bootstrap 5.3.3
Bundle complete! 17 Gemfile dependencies, 93 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.



## Setting up jquery
