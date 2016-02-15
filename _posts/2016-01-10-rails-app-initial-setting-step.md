---
layout: post
title: "rails app initial setting step"
description: ""
category: rails
tags: [rails]
---
{% include JB/setup %}

# edit database.yml

# gemfile

### kaminari
`rails g kaminari:config`  generate config/initializers/kaminari_config.rb
`rails g kaminari:views bootstrap3`   bs theme

### rspec
`rails generate rspec:install`

### simple_form
`rails generate simple_form:install`  install
`rails generate simple_form:install --bootstrap`  theme

### DatabaseCleaner
**spec/rails_helper.rb**
```
config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end

config.around(:each) do |example|
  DatabaseCleaner.cleaning do
    example.run
  end
end
```

### bootstrap-sass
**application.css**
```
@import "bootstrap-sprockets";
@import "bootstrap";
```
**application.js**
```
//= require bootstrap-sprockets
```

### devise
```
rails generate devise:install
rails g devise:views
rails generate devise user
```
**config/environments/development.rb:**
```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

### rails_layout

rails generate layout:install bootstrap3
>>>
remove  app/assets/stylesheets/application.css
create  app/assets/stylesheets/application.css.scss
create  app/assets/stylesheets/1st_load_framework.css.scss
 force  app/assets/javascripts/application.js
append  app/assets/stylesheets/1st_load_framework.css.scss
create  app/views/layouts/application.html.erb
create  app/views/layouts/_messages.html.erb
create  app/views/layouts/_navigation.html.erb
create  app/views/layouts/_navigation_links.html.erb

rails generate layout:devise bootstrap3
>>>
force  app/views/devise/sessions/new.html.erb
force  app/views/devise/passwords/new.html.erb
force  app/views/devise/passwords/edit.html.erb
force  app/views/devise/registrations/edit.html.erb

rails generate layout:navigation
>>>
identical  app/views/layouts/_navigation_links.html.erb
   create  app/views/layouts/_nav_links_for_auth.html.erb
     gsub  app/views/layouts/_navigation.html.erb
   create  spec/features/visitors/navigation_spec.rb
