---
layout: post
title: "ransack tip"
description: ""
category:
tags: ransack rails
---
{% include JB/setup %}

### collection_select
`<%= f.collection_select(:client_id_eq, Client.all, :id, :client_name) %>`
