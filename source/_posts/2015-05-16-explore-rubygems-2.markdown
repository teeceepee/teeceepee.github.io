---
layout: post
title: "Explore RubyGems 2"
date: 2015-05-16 14:42
comments: true
categories: RubyGems
---

# What is a gem

gem 是一种 Ruby 软件包，每个 gem 都有一个名字、版本和平台。名字和版本好理解，平台指的是 CPU 架构、操作系统及版本等信息。

## gem 的目录结构

gem 的目录结构遵循一些约定，如下：

```
% tree freewill
freewill/
├── bin/
│   └── freewill
├── lib/
│   └── freewill.rb
├── test/
│   └── test_freewill.rb
├── README
├── Rakefile
└── freewill.gemspec

```

`lib` 目录里是 gem 的源代码。除此之外，最重要的就是 `.gemspec` 文件。`.gemspec` 文件包含了构建一个 gem 的所有的信息，包括 gem 的名字、版本、平台、包含的文件等。


## Make Your Own Gem

构建一个 gem 可能是理解 gem 的最好方法。下面会从零开始创建一个示例 gem，我给这个 gem 取名为 `factory_boy` 。

首先新建一个目录，下面所有的工作都在该目录内完成。

```
% mkdir factory_boy/
% cd factory_boy
```

一个 gem 肯定要包含一些代码的，按照约定，这些代码应该放到 `lib` 目录中，而且 `lib` 目录中应该有一个和 gem 同名的 `.rb` 文件。

```
% cat lib/factory_boy.rb
module FactoryBoy
  def self.hi
    puts "Hello, I'm a factory boy."
  end
end

```

好，现在 gem 的代码已经有了，还需要一个 `.gemspec` 文件，把它命名为 `factory_boy.gemspec`。

``` ruby factory_boy.gemspec
Gem::Specification.new do |spec|
  spec.name        = 'factory_boy'
  spec.version     = '0.0.1'
  spec.date        = '2015-05-16'
  spec.summary     = "I am a factory boy."
  spec.description = "A factory boy want to meet a factory girl."
  spec.authors     = ["Factory Boss"]
  spec.email       = 'fb@example.com'
  spec.files       = ["lib/factory_boy.rb"]
  spec.homepage    = 'http://fb.example.com'
  spec.license     = 'MIT'
end
```

`factory_boy.gemspec` 文件描述了该 gem 的许多信息，比较重要的是通过 `spec.files = ...` 指定 gem 包含的文件，如果没有指定该项，构建出的 gem 是一个不包含任何文件的空 gem。

现在就可以构建一个最简单的 gem 了, `gem build` 命令可以方便的完成这个操作，将对应的 `.gemspec` 文件作为参数即可。

``` bash
% gem build ./factory_boy.gemspec
  Successfully built RubyGem
  Name: factory_boy
  Version: 0.0.1
  File: factory_boy-0.0.1.gem

```

`gem build` 命令会将构建好的 gem 放在当前目录，文件名中包含了 gem 的名字和版本。要测试这个 gem 是否能正常工作，需要先安装，然后就可以用 irb 或其他方式测试了。

``` bash
% gem install ./factory_boy-0.0.1.gem
Successfully installed factory_boy-0.0.1
1 gem installed


% irb
irb(main):001:0> require 'factory_boy'
=> true
irb(main):002:0> FactoryBoy.hi
Hello, I'm a factory boy.
=> nil
```

看上去不错，如果想要将自己编写的 gem 发布到 rubygems.org 上，需要用到 `gem push` 命令。

## 更常见的目录结构

通常一个 gem 会包含多个文件，推荐的做法是在 `lib` 目录下建立一个与 gem 同名的目录，文件都放到该目录内。使用 `require` 时会在 `lib` 目录搜索文件，所以一般 `lib` 目录下只有一个与 gem 同名的 `.rb` 文件。我们可以把刚才的 gem 改成如下的目录结构：

``` bash
% tree lib
lib
├── factory_boy
│   ├── hi.rb
│   └── version.rb
└── factory_boy.rb
```

文件内容改为如下：

``` ruby factory_boy/hi.rb
module FactoryBoy
  def self.hi
    puts "Hello, I'm a factory boy."
  end
end
```

``` ruby factory_boy/version.rb
module FactoryBoy
  VERSION = '0.0.2'
end
```

``` ruby factory_boy.rb
require 'factory_boy/hi'
require 'factory_boy/version'

module FactoryBoy
end
```

把 gem 的实现都放在 `lib/factory_boy` 目录中，然后在 `factory_boy.rb` 中引入这些文件。注意 `factory_boy.rb` 中的 `require` 方法，RubyGems 会正确设置 $LOAD_PATH，确保能找到 `lib/factory_boy` 目录下的文件。

改好之后需要再次测试一下，这次换另一种测试方法，不需要安装这个 gem。当然在这之前需要用 `gem uninstall factory_boy` 卸载之前安装的 gem。

``` bash
% irb -Ilib -rfactory_boy
irb(main):001:0> FactoryBoy::VERSION
=> "0.0.2"
irb(main):002:0> FactoryBoy.hi
Hello, I'm a factory boy.
=> nil
```
上面的 irb 命令会将 `lib` 目录加入 $LOAD_PATH，然后执行 `require 'factory_boy'`，因此进入 irb 就可以直接使用 `FactoryBoy` 这个模块了。

测试看上去没有问题，这时可以构建新的 gem 了。不过构建之前需要对 `.gemspec` 文件做一些修改，因为增加了新的源文件，所以 `spec.files` 这一项必须要修改。每次增加或删除文件都要修改 `.gemspec` 文件，不仅麻烦，而且容易忘记修改。使用 `Dir.[]` 方法是一个不错的选择。修改后的 `.gemspec` 文件如下：

``` ruby factory_boy.gemspec
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'factory_boy/version'

Gem::Specification.new do |spec|
  spec.name        = 'factory_boy'
  spec.version     = FactoryBoy::VERSION
  spec.date        = '2015-05-16'
  spec.summary     = "I am a factory boy."
  spec.description = "A factory boy want to meet a factory girl."
  spec.authors     = ["Factory Boss"]
  spec.email       = 'fb@example.com'
  spec.files       = Dir['lib/**/*']
  spec.homepage    = 'http://fb.example.com'
  spec.license     = 'MIT'
end
```

主要有两方面的修改，第一，使用 `Dir['lib/**/*']` 获得 `lib` 目录下的所有文件，这样增加或删除文件时就不需要修改 `.gemspec` 文件了。第二，gem 的版本通过 `factory_boy/version.rb` 文件获得，这样保证修改版本时只需修改一处代码即可，满足 DRY 的原则，而且这也是多数 gem 采用的方式。

## 增加可执行文件

gem 除了能提供 Ruby 代码，还可以提供一个或多个可执行文件。比如常见的 `rake` `bundle` `rails` 命令都是对应的 gem 提供的可执行文件。向一个 gem 中添加可执行文件非常简单，只要在 gem 目录中创建可执行文件，然后在 `.gemspec` 文件中声明就好了。构建 gem 时，会从 `spec.bindir` 对应的目录中查找 `spec.executables` 中对应的文件作为可执行文件。 `spec.bindir` 的默认值是 `bin` ，但是 Bundler 1.8 [建议][bundler_moves_bins_to_exe]改为 `exe`，因为 bin 目录是存放 Bundler 的 binstubs 的目录。下面的例子遵循 Bundler 的建议，创建 `exe/factory_boy` 文件作为可执行文件，同时修改 `.gemspec` 文件。

``` ruby exe/factory_boy
#!/usr/bin/env ruby

require 'factory_boy'

FactoryBoy.hi
```

``` ruby factory_boy.gemspec
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'factory_boy/version'

Gem::Specification.new do |spec|
  spec.name        = 'factory_boy'
  spec.version     = FactoryBoy::VERSION
  spec.date        = '2015-05-16'
  spec.summary     = "I am a factory boy."
  spec.description = "A factory boy want to meet a factory girl."
  spec.authors     = ["Factory Boss"]
  spec.email       = 'fb@example.com'
  spec.files       = Dir['lib/**/*']
  spec.bindir      = 'exe'
  spec.executables = ['factory_boy']
  spec.homepage    = 'http://fb.example.com'
  spec.license     = 'MIT'
end
```

经过这样的修改，`gem build` 构建出的 gem 就包含了可执行文件，在安装后就可以在命令行中使用了。

``` bash
% gem build factory_boy.gemspec
% gem install ./factory_boy-0.0.2.gem
% factory_boy
Hello, I'm a factory boy.
```

## 增加依赖 gem

gem 之间是有依赖关系的，如果自己编写的 gem 需要依赖其他 gem，只需在 `.gemspec` 文件中声明依赖的 gem 名称和版本即可。

``` ruby
  spec.add_dependency 'factory_girl', '~> 4.5'
```

这样在构建 gem 时依赖关系会保存在 gem 包中，安装时会保证依赖的 gem 已经安装。

## gem 包的文件格式

gem 包其实是一个 tar 包，用下面的命令可以查看。

``` bash
% file factory_boy-0.0.2.gem
factory_boy-0.0.2.gem: POSIX tar archive
```

既然是 tar 包，就可以展开查看一下里面的内容。

``` bash
% mkdir tmp
% cd tmp

% tar -xvf ../factory_boy-0.0.2.gem
x metadata.gz
x data.tar.gz
x checksums.yaml.gz
```

可以看到，tar 包里面有三个文件，这三个文件都是用 gzip 压缩过的，其中 `checksum.yaml.gz` 的内容是另外两个压缩文件的校验和，`metadata.gz` 的内容是该 gem 的一些元信息，包括 gem 名称、版本、依赖关系等。`data.tar.gz` 是所有 gem 的文件，安装 gem 时会把这些文件安装到对应路径。

## 参考

[make_your_own_gem][]

[bundler_moves_bins_to_exe][]


[make_your_own_gem]: http://guides.rubygems.org/make-your-own-gem/
[bundler_moves_bins_to_exe]: http://bundler.io/blog/2015/03/20/moving-bins-to-exe.html
