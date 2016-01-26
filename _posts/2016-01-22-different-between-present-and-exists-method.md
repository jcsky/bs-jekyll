---
layout: post
title: "different between present? and exists? method"
description: ""
category:
tags: []
---
{% include JB/setup %}

different between present? and exists?
1.
```
User.all.exists => true
User.first.exists => no method error

User.all.present? => true
User.first.present? => true
```

2.exists? have better performance
when you call present? it initializes ActiveRecord for each record found(!), while exists? does not

3.if data is already, use present?, do not exec sql query, but exists? will do sql query
如果資料已經撈出來了 view 上要用，那用 present 比較快，不需要再查一次
exists? 則一定會用 SQL 查詢一次，用 SELECT 1 來查
