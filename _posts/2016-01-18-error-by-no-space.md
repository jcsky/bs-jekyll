---
layout: post
title: "error by no space"
description: "rails production server no space, clean log file"
category: rails
tags: [logrotate rails]
---
{% include JB/setup %}

I used **airbrake + errbit** server to get error message

recently, my rails production server got many errors like `MESSAGE
100.0%	Errno::ENOSPC: No space left on device @ io_write - /tmp/open-uri20160118-27742-1yaoq5`

use `df -h` command check disk space
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      7.8G  7.4G     0 100% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev            2.0G   12K  2.0G   1% /dev
tmpfs           396M  340K  395M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            2.0G     0  2.0G   0% /run/shm
none            100M     0  100M   0% /run/user
```

notice **Use% => 100%**

I can not deploy
I can not even touch tab to autocomplete linux command

check my log file dir
```
total 2.9G
drwxrwxr-x 2 apps     apps 4.0K Nov 30 09:52 ./
drwxrwxr-x 8 apps     apps 4.0K Jul 27 17:41 ../
-rw-r--r-- 1 www-data root 2.8G Jan 18 09:35 access.log
-rw-rw-r-- 1 apps     apps    0 Jun 26  2015 development.log
-rw-r--r-- 1 www-data root 5.6M Jan 18 09:35 error.log
-rw-r--r-- 1 apps     apps  39M Jan 18 09:02 newrelic_agent.log
-rw-rw-r-- 1 apps     apps  59M Jan 18 01:41 sidekiq.log
-rw-rw-r-- 1 apps     apps    0 Nov 30 09:52 tw-production.log
```

so I delete access.log(2.8G)

after deleted log file, the issue was not solved, because the file still open

we need to restart nginx `sudo service nginx restart`

### use logrotate to regular cleaning log file

edit **/etc/logrotate.conf** and add following configuration

```
/home/apps/appname/current/log/access.log {
    monthly
    missingok
    rotate 6
    compress
    delaycompress
    notifempty
    copytruncate
}
```

[copy from ihower blog](https://ihower.tw/blog/archives/3565)
daily 表示每天整理，也可以改成 weekly 或 monthly
missingok 表示如果找不到 log 檔也沒關係
rotate 7 表示保留七份
compress 表示壓縮起來，預設用 gzip
delaycompress 表示延後壓縮直到下一次 rotate
notifempty 表示如果 log 檔是空的，就不 rotate
copytruncate 先複製 log 檔的內容後，在清空的作法，因為有些程式一定 log 在本來的檔名，例如 rails。另一種方法是 create

run logrotate now by `sudo /usr/sbin/logrotate -f /etc/logrotate.conf`, and check your log file/dir
