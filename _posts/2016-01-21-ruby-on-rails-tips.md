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

### reduct, inject
`[100, 200, 1000].reduce :+` # => 1300

`[100, 200, 1000].reduce :*` # => 20000000

`[100, 200, 1000].inject {|sum, n| sum + n }` #=> 1300

# About SQL

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


### Updating multiple records:
```
people = { 1 => { "first_name" => "David" }, 2 => { "first_name" => "Jeremy" } }
Person.update(people.keys, people.values)
```

# sample code
[智付寶 pay2go by ihower](https://github.com/ihower/rails-exercise-ac7/pull/1)
