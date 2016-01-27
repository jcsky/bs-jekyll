---
layout: post
title: "rails export csv file"
description: ""
category:
tags: []
---
{% include JB/setup %}

**/config/application.rb**
```ruby
require 'csv'
```

**model.to_csv method**
```ruby
def self.to_csv
  attributes = %w{id email name}

  CSV.generate(headers: true) do |csv|
    csv << attributes #header

    all.each do |user|
      csv << attributes.map{ |attr| user.send(attr) }
    end
  end
end
```

**controller**
```ruby
respond_to do |format|
  format.html
  format.csv { send_data @users.to_csv, filename: "users-#{Date.today}.csv" }
end
```

**or more simple**
```ruby
send_data User.to_csv, :type => 'csv', :disposition => "attachment; filename=email_list.csv"
```

**by the way send text use send_data**
```ruby
@content = "Hello World"
send_data @content,
  :type => 'text',
  :disposition => "attachment; filename=your_file_name.txt"
```

references
[send_data, send_file](http://api.rubyonrails.org/classes/ActionController/DataStreaming.html)
[RailsCasts](http://railscasts.com/episodes/362-exporting-csv-and-excel?view=asciicast)
