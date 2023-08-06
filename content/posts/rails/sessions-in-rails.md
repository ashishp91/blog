---
title: "Sessions in Rails"
date: 2023-08-06T17:42:55+05:30
draft: false
tags: ["Ruby", "Rails"]
---

Sessions provide a way to store any kind of data across requests. This could be user info, preferences, cart items etc.

Using sessions is fairly easy, Rails provides a session variable which is a dictionary with key-value pairs.

```ruby
# Writes login of the user in the session
session[:user_login] = user.login

# Reading from session
user_login = session[:user_login]
```

Since HTTP is a stateless protocol, sessions comes in handy in retaining info across subsequent requests.

## Using session in controllers

In Rails generally, sessions are used in controllers as they are the entrypoint for all incoming HTTP requests. For eg. when we're creating a session we want to store the user id in the session:

```ruby
class SessionsController < ApplicationController
    def create
        ...
        session[:current_user_id] = @user.id
        ...
    end
end
```

## Using req variable to access session

Session can also be accessed via the request object `req` . One can call the session method on the req object, here's an example of using req in routes to apply a constraint on the basis of the user name:

```ruby
get 'sessions', to: 'projects#sessions', constraints: ->(req) {
  req.session[:name] = 'test'
  true
}, as: 'sessions'
```
## But how exactly session variable works ?

By default the data in sessions are stored in browser as a cookie. The Rails server at the very first time sends back the data stored in the session which is then saved by the browser as a cookie.

Afterwards, in subsequent requests the browser the sends the cookie with each request as header. Rails server reads that and sets it in the session variable.

Sessions can be stored in other places as well like database or memcache or redis, Rails provides following stores where session data can persist:

- `CookieStore`: Store all data inside a cookie, in addition to the session ID

- `CacheStore`: Store data inside the Rails cache

- `ActiveRecordStore`: Store the data in the database

- `MemCacheStore`: Use a Memcached cluster of servers

One interesting thing to note is that **Sessions are lazily loaded**. If you don't access sessions at all, they will not be loaded. Hence, you don't have to disable sessions if you aren't using them.
