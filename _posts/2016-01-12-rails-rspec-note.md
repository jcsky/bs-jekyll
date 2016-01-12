---
layout: post
title: "rails rspec note"
description: ""
category:
tags: []
---
{% include JB/setup %}

讓測試稍後再測（待辦事項）

耶！待辦事項。太棒了！一個俏皮的 RSpec 功能，讓妳可以把特定的測試標註為“待辦”（pending）。

怎麼樣把一個測試變成待辦事項呢？

沒有 do 與 end 的 it：

it "has a name"
或是在 it 前面加上 x：

describe Idea do
  xit "has a name" do
    ...
  end
end
或者是把 it 換成 pending 也可以哦！

describe Idea do
  pending "has a name" do
    ...
  end
end

source <http://railsgirls.tw/testing-rspec/>
