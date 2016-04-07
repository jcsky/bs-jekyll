---
layout: post
title: "rake cors gem"
description: ""
category:
tags: [cors rails]
---
{% include JB/setup %}
```ruby
#gemfile
group :development do
  gem "rack-cors"
end

#config/application.rb
require 'rack'
require 'rack/cors'

class Application < Rails::Application
  #...
  config.middleware.insert_before 0, "Rack::Cors", :debug => true, :logger => (-> { Rails.logger }) do
    allow do
      origins '*'

      resource '/cors',
        :headers => :any,
        :methods => [:post],
        :credentials => true,
        :max_age => 0

      resource '*',
        :headers => :any,
        :methods => [:get, :post, :delete, :put, :patch, :options, :head],
        :max_age => 0
    end
  end
end
```
