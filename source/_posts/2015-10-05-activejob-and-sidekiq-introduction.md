---
layout: post
title: "Active Job 和 Sidekiq 简介"
date: 2015-10-05 20:50
comments: true
categories: ActiveJob Sidekiq
---

## Active Job

Rails 4.2 引入了异步任务框架 [Active Job][active_job]。在 Active Job 出现之前，已经有许多与 Rails 配套的异步任务框架了，比如 Resque、Sidekiq 等。Rails 官方 引入 Active Job 的目的并不是取代已有的异步任务框架，而是构建一层通用的异步任务接口，通过适配器对接不同的后端。这样，异步任务使用 Active Job 提供的接口编写，后端可以使用不同的框架执行，有点类似 ActiveRecord 与 MySQL、PostgreSQL 的关系。

下面是一个简单的例子：

``` ruby
# app/jobs/guests_cleanup_job.rb

class GuestsCleanupJob < ActiveJob::Base
  queue_as :default

  def perform(*args)
    puts 'start guests cleanup...'
    sleep(10)
    puts 'stop guests cleanup.'
  end
end
```

## Active Job 与 Sidekiq 集成

根据 Rails Guide 中 [Active Job Basics][guide] 的说明，如果没有设置适配器，任务会立即执行。因此，想要实现任务的异步执行，必须集成一个后端。因为我比较熟悉 Sidekiq，所以选择 Sidekiq 作为后端。

首先在配置中指定使用的后端适配器：

``` ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # Be sure to have the adapter's gem in your Gemfile and follow
    # the adapter's specific installation and deployment instructions.
    config.active_job.queue_adapter = :sidekiq
  end
end
```

正如上面注释中写的，Active Job 只提供了设置后端适配器的选项，其余的设置都要参考 Sidekiq 本身的配置了。首先在 `Gemfile` 中加入 Sidekiq：

``` ruby
gem 'sidekiq'
```

执行 `bundle` 后就可以用 `bundle exec sidekiq` 启动 Sidekiq 了。

## Sidekiq 配置

### Web UI

Sidekiq 有一个 Web 界面，可以比较方便的查看任务状态。首先在 `Gemfile` 中加入 Sinatra。

``` ruby
# Gemfile

# if you require 'sinatra' you get the DSL extended to Object
gem 'sinatra', :require => nil
```

然后在 `config/routes.rb` 中挂载到一个地址，其中 `require 'sidekiq/web'` 是必需的。

``` ruby
# config/routes.rb

Rails.application.routes.draw do
  # ...
  require 'sidekiq/web'
  mount Sidekiq::Web, at: '/sidekiq'
  # ...
end
```

`bundle` 后重启 Rails 服务器就可以在 `localhost:3000/sidekiq` 看到 Sidekiq 的 Web 界面了。

### Redis 设置

Sidekiq 依靠 Redis 实现异步功能，可以对 Redis 做一些设置。

``` ruby
# config/initializers/sidekiq.rb

url = 'redis://localhost:6379/1'
redis_config = {
  url: url,
  namespace: 'sidekiq',
}

Sidekiq.configure_server do |config|
  config.redis = redis_config
end

Sidekiq.configure_client do |config|
  config.redis = redis_config
end
```

上面的配置指定了 Sidekiq 连接 Redis 使用的 URL 以及创建 key 时的前缀。URL 除了主机名和端口号，还可以指定一个数据库号，这个是 Redis 提供的数据隔离的一种方式。比如开发环境使用 0 号数据库，而测试环境使用 1 号数据库。`namespace` 也是数据隔离的一种方式，依赖 `redis-namespace`gem 实现，通过在 key 加上相同的前缀可以方便查看哪些 key 是 Sidekiq 创建和使用的。下面是 redis-cli 的输出。

```
% redis-cli
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> KEYS *
1) "sidekiq:stat:processed"
2) "sidekiq:stat:processed:2015-10-05"
3) "sidekiq:queues"
127.0.0.1:6379[1]>
```

上面的配置中出现了 `configure_server` 和 `configure_client` 两部分，是因为 Sidekiq 是 C/S 架构的。从 Redis 的角度看，向 Redis 队列中增加任务的是客户端，通常是 Puma、Unicorn 等 Rails 进程，从 Redis 队列中取任务的是服务端，一般就是 `bundle exec sidekiq` 启动的进程。

### sidekiq.yml

`sidekiq` 命令有许多选项可以设置，下面是 `bundle exec sidekiq -h` 的输出：

```
% bundle exec sidekiq -h
2015-10-05T14:10:11.607Z 29897 TID-ovozeltdo INFO: sidekiq [options]
    -c, --concurrency INT            processor threads to use
    -d, --daemon                     Daemonize process
    -e, --environment ENV            Application environment
    -g, --tag TAG                    Process tag for procline
    -i, --index INT                  unique process index on this machine
    -q, --queue QUEUE[,WEIGHT]       Queues to process with optional weights
    -r, --require [PATH|DIR]         Location of Rails application with workers or file to require
    -t, --timeout NUM                Shutdown timeout
    -v, --verbose                    Print more verbose output
    -C, --config PATH                path to YAML config file
    -L, --logfile PATH               path to writable logfile
    -P, --pidfile PATH               path to pidfile
    -V, --version                    Print version and exit
    -h, --help                       Show help
```

这些选项同时也可以写到 `config/sidekiq.yml` 配置文件中，比如：

``` yml
# config/sidekiq.yml

---
:daemon: true
:verbose: true
:pidfile: ./tmp/pids/sidekiq.pid
:logfile: ./log/sidekiq.log
```

上面的配置文件告诉 Sidekiq 以守护进程方式运行，输出更多日志，pid 文件和日志文件的路径。

### 结束 Sidekiq 守护进程

如果以守护进程方式运行，除了用 `ps` 和 `kill` 命令结束进程，还可以用 Sidekiq 提供的 `sidekiqctl` 命令。

```
% sidekiqctl stop tmp/pids/sidekiq.pid
Sidekiq shut down gracefully.
```

`sidekiqctl` 还有一个子命令 `quiet`，参考 [Sidekiq signals][sidekiq_signals]。

## 参考

[Active Job][active_job]

[Active Job guide][guide]

[Sidekiq][sidekiq]

[Sidekiq client server][sidekiq_client_server]

[Sidekiq signals][sidekiq_signals]

[active_job]: https://github.com/rails/rails/tree/master/activejob
[guide]: http://guides.rubyonrails.org/active_job_basics.html
[sidekiq]: https://github.com/mperham/sidekiq
[sidekiq_client_server]: https://github.com/mperham/sidekiq/issues/638
[sidekiq_signals]: https://github.com/mperham/sidekiq/wiki/Signals
