---
layout: post
title: "used linux command"
description: ""
category: linux
tags: [linux tail]
---
{% include JB/setup %}  

# tail
- tail file: 預設輸出最後10行
- tail -n：--lines=N 指定輸出檔案最後 "N" 行。
- tail -f：持續顯示檔案更新的資訊。
- tail +n：從第n 行開始顯示到檔案結尾。


# ps (process)
- ps auxwww > /tmp/my_ps
- ps aux

# top (cpu and memory)
- top -o %MEM
- top -o %CPU
