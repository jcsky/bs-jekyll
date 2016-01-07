---
layout: post
title: "set and install sidekiq + redis on production env"
description: ""
category:
tags: []
---
{% include JB/setup %}

##### run sidekiq on background job
`bundle exec sidekiq -d -L log/sidekiq.log -C config/sidekiq.yml -e production`

- -d, Daemonize process
- -L, path to writable logfile
- -C, path to YAML config file
- -e, Application environment

##### sidekiq restart
`cap production sidekiq:start`
