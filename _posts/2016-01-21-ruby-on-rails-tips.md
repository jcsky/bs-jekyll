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



### self join
```
create_table "posts", :force => true do |t|
    t.string   "title"
    t.text     "content"
    t.integer  "parent_post_id"
    t.datetime "created_at",     :null => false
    t.datetime "updated_at",     :null => false
end
```
```
class Post < ActiveRecord::Base
  has_many :replies, :class_name => "Post",
    :foreign_key => "parent_post_id"
  belongs_to :parent_post, :class_name => "Post",
    :foreign_key => "parent_post_id"
end
```

### sort by calculated
`scope :sort_by_pledged_percent, -> { select('*, (pledged / goal) as sort_value').order('sort_value DESC') }`
