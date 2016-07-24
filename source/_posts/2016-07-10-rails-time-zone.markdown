---
layout: post
title: "Rails time zone"
date: 2016-07-10 14:22
comments: true
categories:
---

前几天发现自己写的 Rails 项目数据库里的时间不对，与实际时间相差了八个小时。作为一个生活在东八区的人，很容易理解这是一个与时区相关的问题。

首先检查 Rails 的配置，发现没有与时区相关的配置，说明这是 Rails 的默认行为。查了一些资料后，慢慢理解了 Rails 是如何处理时区的。

首先是如何向数据库中插入时间。通过数据库迁移生成的时间和日期类型的字段是没有时区信息的。`created_at` 在 PostgreSQL 中是 `timestamp without time zone` 类型的，在 MySQL 中是 `datetime` 类型的，显然都是不带时区信息的。Rails 默认在数据库中保存 UTC 时间，也可以通过修改配置让数据库中保存的是本地时间。

``` ruby
# config/application.rb

# default, store utc time
config.active_record.default_timezone = :utc
# store local time
config.active_record.default_timezone = :local
```

到这里就明白为什么数据库里的时间与实际时间相差了八个小时了。所谓的“实际时间”就是本地时间，北京时间，东八区时间，对应的 UTC 时间就是减掉 8 个小时的时间。

```
# Rails console
[5] pry(main)> Bar.find(1).created_at
Bar Load (0.3ms)  SELECT  "bars".* FROM "bars" WHERE "bars"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 CST +08:00

# PostgreSQL console
pg_development> select created_at from bars where id = 1;
+----------------------------+
| created_at                 |
|----------------------------|
| 2016-06-29 15:15:25.769707 |
+----------------------------+

```

可以看到 Rails 自动对数据库里时间做了处理，返回的时间是带有时区信息的。

Rails 可以从哪些地方获取到时区的信息呢？有两种方式。

### 操作系统

第一种是直接从操作系统获取。操作系统都是有时区配置的，Ruby 的 `Time` 和 `DateTime` 类可以获取到操作系统的时区。下面几行命令的输出可以看到时区的影响。

```
% ll /etc/localtime
lrwxr-xr-x  1 root  wheel    33B  7  9 18:08 /etc/localtime -> /usr/share/zoneinfo/Asia/Shanghai

[90] pry(main)> Time.now
=> 2016-07-10 15:12:52 +0800
```

```
% ll /etc/localtime
lrwxr-xr-x  1 root  wheel    30B  7 10 16:13 /etc/localtime -> /usr/share/zoneinfo/Asia/Tokyo

[93] pry(main)> Time.now
=> 2016-07-10 16:13:57 +0900
```

当 OS X 的时区从东八区改为东九区时，`Time.now` 返回的时间也相依的发生变化。其实这种获取时区的方式与 Rails 无关，更准确的说这是 Ruby 获取时区的方式。

### Rails
第二种方式是在是通过配置文件设置 Rails 应用的时区。

``` ruby
# config/application.rb
config.time_zone = 'UTC'

config.time_zone = 'Beijing'

config.time_zone = 'Tokyo'
```

`config.time_zone` 设置 Rails 默认使用的时区，默认值是 'UTC'，它会影响到 Rails 里时间的显示，比如：

```
# config.time_zone = 'UTC'
[11] pry(main)> Bar.first.created_at
  Bar Load (1.2ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 15:15:25 UTC +00:00

# config.time_zone = 'Beijing'
[1] pry(main)> Bar.first.created_at
  Bar Load (0.5ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 CST +08:00
```

可以看到，相同的时间，因为默认时区的不同，显示是不同的。如果我们把 UTC 时间当做绝对时间，设置时区只是让 Rails 把绝对时间显示为对应时区的时间。除了默认的时区设置，还有其他设置时区的方式。

#### Time#zone=

Rails 扩展了 `Time` 类，增加了 `zone=` 等方法，`config.time_zone = 'Beijing'` 实际上就等价于 `Time.zone = 'Beijing'`，只不过使用 `Time#zone=` 可以重设默认时区。

```
# config.time_zone = 'Beijing'
[1] pry(main)> Bar.first.created_at
  Bar Load (0.4ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 CST +08:00

[2] pry(main)> Time.zone = 'Beijing'
=> "Beijing"
[3] pry(main)> Bar.first.created_at
  Bar Load (0.4ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 CST +08:00

[4] pry(main)> Time.zone = 'Singapore'
=> "Singapore"
[5] pry(main)> Bar.first.created_at
  Bar Load (0.3ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 SGT +08:00

[6] pry(main)> Time.zone = 'Tokyo'
=> "Tokyo"
[7] pry(main)> Bar.first.created_at
  Bar Load (0.3ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Thu, 30 Jun 2016 00:15:25 JST +09:00
```

#### ActiveSupport::TimeWithZone#in_time_zone

`Time#zone=` 修改时区后会影响到所有时间的显示，`in_time_zone` 只针对一个对象，不会影响其他的时间对象。

```
[1] pry(main)> t = Bar.first.created_at
  Bar Load (0.5ms)  SELECT  "bars".* FROM "bars" ORDER BY "bars"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> Wed, 29 Jun 2016 23:15:25 CST +08:00
[2] pry(main)> t.in_time_zone('UTC')
=> Wed, 29 Jun 2016 15:15:25 UTC +00:00
[3] pry(main)> t.in_time_zone('Singapore')
=> Wed, 29 Jun 2016 23:15:25 SGT +08:00
[4] pry(main)> t.in_time_zone('Tokyo')
=> Thu, 30 Jun 2016 00:15:25 JST +09:00
[5] pry(main)> t
=> Wed, 29 Jun 2016 23:15:25 CST +08:00

```

## 时区的影响

目前看来，不管如何设置时区，貌似只会影响时间的显示，毕竟修改时区不会改变绝对时间，但某些场景中时区就显得举足轻重了。比如“每天零点重新计算用户活跃度”，“每周一上午八点发送推荐邮件”，我们发现如果不指明时区，“零点”和“上午八点”这样的时间是无意义的。在类似的场景中，时区是必须提供的一个信息。在一些国际化产品中，用户来自全世界各地，在查看报表等信息时，用户当然希望显示的是自己所在时区的时间。一种解决方式是在用户表中增加一个字段来保存用户的时区，然后用 `in_time_zone` 或下面的 `around_action` 将时间显示为用户所在时区的本地时间。

``` ruby
# http://api.rubyonrails.org/classes/Time.html#method-c-zone-3D
class ApplicationController < ActionController::Base
  around_filter :set_time_zone

  def set_time_zone
    if logged_in?
      Time.use_zone(current_user.time_zone) { yield }
    else
      yield
    end
  end
end
```

## 参考

[The Exhaustive Guide to Rails Time Zones](http://danilenko.org/2012/7/6/rails_timezones/)

[It's About Time (Zones)](https://robots.thoughtbot.com/its-about-time-zones)

[http://stackoverflow.com/questions/6118779/how-to-change-default-timezone-for-active-record-in-rails](http://stackoverflow.com/questions/6118779/how-to-change-default-timezone-for-active-record-in-rails)

[http://api.rubyonrails.org/classes/Time.html#method-c-zone-3D](http://api.rubyonrails.org/classes/Time.html#method-c-zone-3D)
