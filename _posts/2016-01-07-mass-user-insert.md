---
layout: post
title: "mass user insert"
description: "mass user insert in one sql"
category: rails
tags: [rails ruby sql]
---
{% include JB/setup %}

[Xdite rails massive data](http://blog.xdite.net/posts/2012/08/22/rails-with-massive-data)

SolveIssue Sample

```
  task :mass_user => :environment do
    CONN = ActiveRecord::Base.connection
    inserts = []
    time = Time.now.to_s(:db)
    a = User.all.count
    (a..a + 100).each do |n|
      # 避免email重覆
      email = "a#{n}@email.com"
      inserts.push %Q{('#{email}', '12345678', 0, '#{country.sample}', '#{Faker::Avatar.image}', '#{Faker::Internet.user_name}', '#{time}', '#{time}')}
    end
    sql = "INSERT INTO users (email, encrypted_password, role, country, fb_image, name, created_at, updated_at) VALUES #{inserts.join(", ")}"
    begin
      CONN.execute sql
      puts '新增 1萬筆 User'
    rescue
      puts '失敗'
    end
  end
```
