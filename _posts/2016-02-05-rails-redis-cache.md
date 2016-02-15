---
layout: post
title: "rails redis cache"
description: ""
category:
tags: []
---
{% include JB/setup %}

gem 'redis-rails'
gem 'redis-rack-cache' # optional
[參考](http://translate17.com/article/1417)

\# config/environments/production

```ruby
config.cache_classes = true

config.cache_store = :redis_store, 'redis://xxx.cache.amazonaws.com:6379/0', {
  expires_in: 90.minutes,
  namespace: 'appname_cache'
}
```
