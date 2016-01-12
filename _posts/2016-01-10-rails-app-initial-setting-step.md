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
