---
layout: post
title: "rails app initial setting step"
description: ""
category: rails
tags: [rails]
---
{% include JB/setup %}

# gemfile

**kaminari**
`rails g kaminari:config`  generate config/initializers/kaminari_config.rb
`rails g kaminari:views bootstrap3`   bs theme

**rspec**
`rails generate rspec:install`

**simple_form**

`rails generate simple_form:install`  install
`rails generate simple_form:install --bootstrap`  theme

**DatabaseCleaner**
`spec/rails_helper.rb`
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
