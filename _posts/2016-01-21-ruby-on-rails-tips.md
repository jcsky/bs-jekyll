---
layout: post
title: "Ruby on Rails tips"
description: ""
category:
tags: []
---
{% include JB/setup %}

### array group count to hash
`x = Hash[arr.uniq.map{ |i| [i, arr.count(i)] }]`

### group_by_day
`events_count = Event.group_by_day(:created_at).count`
