---
layout: post
title: "active record note"
description: ""
category:
tags: []
---
{% include JB/setup %}

**sort by devision**
`scope :sort_by_pledged_percent, -> { select('*, (pledged / goal) as s').order('s DESC') }`
