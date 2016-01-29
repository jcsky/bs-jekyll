---
layout: post
title: "Ruby Percent String"
description: ""
category:
tags: []
---
{% include JB/setup %}


### %i  Array of Symbols
```
%i{ cat sheep bear }
=> [:cat, :sheep, :bear]
```

### %w  Array of Strings
```
%w{ cat sheep bear }
=> ["cat", "sheep", "bear"]
```

### %q  String
```
%q{ cat sheep bear }
=> " cat sheep bear "
```

### %s  Symbol
```
%s{ cat sheep bear }
=> :" cat sheep bear "
```

### %x  Backtick (capture subshell result)
使用方法執行一段 shell script 並回傳標準輸出的結果
```
%x(echo foo:#{'Foo'})
=> "foo:Foo\n"
```

### %r  Regular Expression
產生正規表示式
```
%r(/home/#{'Foo'})
=> /\\/home\\/Foo/
```
