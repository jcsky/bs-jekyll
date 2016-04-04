---
layout: post
title: "active_admin gem setting"
description: ""
category:
tags: [rails gem active_admin]
---
{% include JB/setup %}

### document
https://github.com/activeadmin/activeadmin/tree/master/docs

http://activeadmin.info/docs/0-installation.html#setting-up-active-admin

# install

`gem 'activeadmin'`

```ruby
rails g active_admin:install              # creates the AdminUser class
rails g active_admin:install User         # creates / edits the class for use with Devise
rails g active_admin:install --skip-users # skips Devise install
```

`rake db:migrate`

# generate model
`rails generate active_admin:resource MyModel`

### add default admin user
`AdminUser.create!(:email => 'admin@example.com', :password => 'password', :password_confirmation => 'password')`
