---
layout: post
title: "rspec only run specific test"
description: ""
category:
tags: [rspec]
---
{% include JB/setup %}

In your spec_helper.rb:
```
RSpec.configure do |config|
    config.filter_run focus: true
    config.run_all_when_everything_filtered = true
end
```

and then on your specs:
```
it 'can do so and so', focus: true do
    # This is the only test that will run
end
```

You can also focus tests with 'fit' or exclude with 'xit', like so:
```
fit 'can do so and so' do
    # This is the only test that will run
end
```

[references stackoverflow](http://stackoverflow.com/questions/5069677/how-do-i-run-only-specific-tests-in-rspec)
