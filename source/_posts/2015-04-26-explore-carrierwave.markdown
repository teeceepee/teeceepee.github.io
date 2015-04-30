---
layout: post
title: "Explore CarrierWave"
date: 2015-04-26 20:28
comments: true
categories:
---

[CarrierWave][1] 是一个 Ruby 应用的上传组件，很多 Rails 程序都会用到它。与它类似的 gem 还有 PaperClip、Dragonfly 等。因为遇到过的项目都是用 CarrierWave，所以想从零开始学习一下他的使用方法。在 GitHub 上的 master 分支是一个正在开发的版本，下面用的都是 0.10.0 版本。

在 Rails 项目的 Gemfile 中添加 CarrierWave 并执行 `bundle install` 后就可以使用它了。CarrierWave 提供了一个 generator 来方便的生成 uploader。用法如下：

``` bash
rails generate uploader Avatar
```

上面的命令会生成 `app/uploaders/avatar_uploader.rb` 文件。从文件的路径中可以看出一些 CarrierWave 的约定。所有的 uploader 都放在 `app/uploaders` 目录下，这个目录不是 Rails 默认有的目录，如果不存在 CarrierWave 会自动创建。所有的 uploader 文件名都以 `uploader` 结尾，对应的类名也是 `XyzUploader`。

在 Rails 程序中很少单独使用 uploader，一般都会与 ORM 配合使用。如果使用 ActiveRecord，可以用 `mount_uploader` 将 uploader 与模型关联。比如：

``` ruby
class User < ActiveRecord::Base
  mount_uploader :avatar, AvatarUploader
end
```

`mount_uploader` 的第一个参数对应数据库的一个字符串类型的字段，用于存储上传文件的文件名，第二个参数是使用的 uploader 类名。

默认生成的 uploader 这样就可以使用了，如果发现不能正常使用的话需要重启服务器。但是在 Rails console 里貌似没有自动加载 `app/uploaders` 目录下的文件，这样会出现 `Uninitialized constant ...` 的异常。如果遇到这样的情况又想在 Rails console 中测试，需要在 `config/application.rb` 中将目录加入到自动加载的列表中。

``` ruby
config.autoload_paths += %W(#{config.root}/app/uploaders)
```

这个问题可能是 CarrierWave 的一个 BUG，参考

<http://www.codeomnib.us/rails-4-carrierwave-throws-uninitialized-uploader-console/>
<https://github.com/carrierwaveuploader/carrierwave/issues/399>

## 存储方式

CarrierWave 支持多种存储方式，既可以将文件存储在本地的硬盘中，也可以使用各种云存储。CarrierWave 默认生成的 uploader 中有一行 `storage :file`，表示将文件存在本地硬盘。

## 存储目录

如果使用本地文件的存储方式，需要指定所有上传文件的存储目录。存储目录由 uploader 中的 `store_dir` 指定，默认生成的 uploader 文件中已经有一个可以使用的 `store_dir` 方法：

``` ruby
def store_dir
  "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
end
```

从中可以看出一些 `CarrierWave::Uploader::Base` 类的一些方法。`model` 方法表示该 uploader 对应的模型，`mounted_as` 方法的值就是在模型中使用 `mount_uploader` 的第一个参数，也就是数据库中对应的字段名。

## 扩展名限制

在实际的上传情景中，可能需要对上传文件的扩展名做一些限制，CarrierWave 提供了白名单和黑名单两种方式，两种方式分别通过 `extension_white_list` 和 `extension_black_list` 方法实现。CarrierWave 默认生成的 uploader 在注释中给出了这两个方法的实现方式。

``` ruby
  def extension_white_list
    %w(jpg jpeg gif png)
  end
```

方法只要返回允许或拒绝的扩展名数组即可，而且不区分大小写。数组的元素除了可以是字符串，还可以是正则表达式，比如 `[/jpe?g/, 'gif', 'png']`。

## 文件名

CarrierWave 在保存文件名时会对原始文件名进行修改，默认的行为是只保留英文字符、数字以及 `.-+_` 四个符号，其他字符会被转化为 `_`，实现方式在 CarrierWave 源码的 `lib/carrierwave/sanitized_file.rb` 中：

``` ruby
# lib/carrierwave/sanitized_file.rb

module CarrierWave
  class SanitizedFile
    # ...

    class << self
      attr_writer :sanitize_regex

      def sanitize_regex
        @sanitize_regex ||= /[^a-zA-Z0-9\.\-\+_]/
      end
    end

    # ...

    def sanitize_regexp
      CarrierWave::SanitizedFile.sanitize_regexp
    end

    # ...

    # Sanitize the filename, to prevent hacking
    def sanitize(name)
      name = name.gsub("\\", "/") # work-around for IE
      name = File.basename(name)
      name = name.gsub(sanitize_regexp,"_")
      name = "_#{name}" if name =~ /\A\.+\z/
      name = "unnamed" if name.size == 0
      return name.mb_chars.to_s
    end

    # ...
  end
end
```

从代码中可以看到，默认使用 `/[^a-zA-Z0-9\.\-\+_]/` 对文件名进行处理，匹配的字符被替换为 `_`。同时看到也可以使用其他的正则对文件名进行处理。比如保留原始文件名：

``` ruby
# config/initializers/carrierwave.rb

CarrierWave::SanitizedFile.sanitize_regex = /[^[:word:]\.\-\+]/
```

创建 `config/initializers/carrierwave.rb` 文件，写入这行代码 。这是官方文档提供的正则，需要注意的是，这个正则表达式匹配的是所有不允许的字符。

如果想要自定义上传文件的文件名，需要重写 `filename` 方法。下面的例子参考 CarrierWave 的 Wiki：

``` ruby
class PhotoUploader < CarrierWave::Uploader::Base
  def filename
    "#{secure_token}.#{file.extension}" if original_filename.present?
  end

  protected
  def secure_token
    var = :"@#{mounted_as}_secure_token"
    model.instance_variable_get(var) or model.instance_variable_set(var, SecureRandom.uuid)
  end
end

```

目前还不太明白为什么要把实例变量存在模型对象而不是 uploader 对象中。但是一定要像例子中那样将 `SecureRandom.uuid` 的值用实例变量缓存起来，否则数据库中存储的文件名会与本地硬盘中的文件名不同，因为在处理过程中 `filename` 方法会被多次使用。

需要注意的是 `filename` 不能返回空字符串，我在尝试的过程中又一次不小心返回空字符串，结果出现了奇怪的错误。看了源码后发现 CarrierWave 的处理方式稍微有些问题。首先，对于重写 `filename` 返回空字符串或 nil 的做法 CarrierWave 并没有做检查，直接认为是合法的文件名。其次，CarrierWave 会改变上传文件的权限，目录的默认权限是 `755`，文件的默认权限是 `644`，这本身没问题，不过当文件名是空字符串或 nil 时，CarrierWave 还是会尝试将文件的权限改为 `644`，而这时的路径实际上对应的是一个目录。最终的结果就是将目录的执行权限去掉了，没有执行权限的目录是无法进入的，自然会出现一些奇怪的错误，比如 `Permission Denied` 。与这个问题相关的文件是 `lib/carrierwave/uploader/store.rb`。

## 图片处理

很多时候需要对上传的图片进行裁剪等处理，然后再保存。CarrierWave 通过 MiniMagick 或 RMagick 来实现对图片的处理。官方推荐使用 MiniMagick，下面的例子也都使用 MiniMagick。

如果使用 MiniMagick，需要在 Gemfile 中将 MiniMagick 包含进来。如果修改了 Gemfile 然后再执行 `bundle install`，通常都需要重启服务器，Rails 的自动重新加载对 gem 无效。

将 MiniMagick 加入项目之后，只需将默认生成的 uploader 文件中的一行代码取消注释就可以使用图片处理功能了。

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  # include CarrierWave::MiniMagick
  include CarrierWave::MiniMagick

  # ...
end
```

`CarrierWave::MiniMagick` 这个模块包含了几个用于处理图片的方法：

- `convert` 转换图片格式。
- `resize_to_limit` 改变图片的大小，不超过指定的宽高，也就是只对超过限制的图片处理，不会有小图片变大的情况。
- `resize_to_fit` 改变图片的大小，适应到指定的宽高，小图片会变大，大图片会变小。
- `resize_to_fill` 改变图片的大小，填充到指定的宽高，可能会将比较大的维度裁减掉一部分。
- `resize_and_pad` 与 `resize_to_fit` 类似，不足的部分用指定的颜色填充。
- `manipulate!` 比较底层的方法，可以实现对图片的自定义处理，前面的方法都是用该方法实现的。

以 resize 开头的四个方法只改变图片的大小，不会改变宽高比，所以用这些方法是不会把图片拉伸的。

要在 uploader 中使用这几个方法来处理图片，要用到类方法 `process`，根据注释，`process` 方法本质上是注册一个在文件上传时执行的回调，接受的参数既可以是 uploader 中的方法名或方法名列表，也可以是一个 Hash，Hash 的键是方法名，值是调用方法需要的参数数组。方法的注释和实现如下：

``` ruby
        ##
        # Adds a processor callback which applies operations as a file is uploaded.
        # The argument may be the name of any method of the uploader, expressed as a symbol,
        # or a list of such methods, or a hash where the key is a method and the value is
        # an array of arguments to call the method with
        #
        # === Parameters
        #
        # args (*Symbol, Hash{Symbol => Array[]})
        #
        # === Examples
        #
        #     class MyUploader < CarrierWave::Uploader::Base
        #
        #       process :sepiatone, :vignette
        #       process :scale => [200, 200]
        #       process :scale => [200, 200], :if => :image?
        #       process :sepiatone, :if => :image?
        #
        #       def sepiatone
        #         ...
        #       end
        #
        #       def vignette
        #         ...
        #       end
        #
        #       def scale(height, width)
        #         ...
        #       end
        #
        #       def image?
        #         ...
        #       end
        #
        #     end
        #
        def process(*args)
          new_processors = args.inject({}) do |hash, arg|
            arg = { arg => [] } unless arg.is_a?(Hash)
            hash.merge!(arg)
          end

          condition = new_processors.delete(:if)
          new_processors.each do |processor, processor_args|
            self.processors += [[processor, processor_args, condition]]
          end
        end
```

可以看到如果键是 `:if` 会把它当做判断回调是否执行的条件。

有了 `CarrierWave::MiniMagick` 和 `process`，二者组合起来就可以实现对上传图片的处理了：

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  process convert: 'png'
  process resize_to_limit: [200, 200]
  process resize_to_fit: [200, 200]
  process resize_to_fill: [200, 200]
end
```

`process` 可以注册多个回调，不过多次执行改变图片大小的回调会把图片变成什么样就不清楚了。

其实 `CarrierWave::MiniMagick` 模块中还提供了简化的类方法来实现同样的作用，在网上找到的教程好像都没提到这一点，代码如下：

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  convert 'png'
  resize_to_limit 200, 200
  resize_to_fit 200, 200
  resize_to_fill 200, 200
end
```

## 多版本

CarrierWave 支持将上传的文件处理成多个版本分别保存，比如生成上传图片的缩略图。在 uploader 中使用 `version` 方法创建一个新版本，版本可以嵌套。

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  version :thumb

  version :foo do
    version :bar
    version :baz
  end
end
```

上面的代码一共创建了四个版本，分别是 `thumb`，`foo`，`foo_bar`，`foo_baz`。创建新版本并不会影响默认的文件版本，所以使用上面的 uploader 每次上传会在本地硬盘存储五个文件。

单纯创建版本只会生成多个内容相同的文件，并没有什么作用，只有搭配使用 `process` 方法才能发挥它的作用。例子如下：

``` ruby
class AvatarUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  process resize_to_limit: [800, 800]

  version :thumb do
    process resize_to_fill: [50, 50]
  end
end
```

这样每次上传会生成两个文件，默认版本大小限制在 800x800 像素以内，`thumb` 版本处理为 50x50 像素。

## 参考

[CarrierWave][1]

[Uninitialized uploader][2]

[CarrierWave issue #399][3]


[1]: https://github.com/carrierwaveuploader/carrierwave
[2]: http://www.codeomnib.us/rails-4-carrierwave-throws-uninitialized-uploader-console/
[3]: https://github.com/carrierwaveuploader/carrierwave/issues/399
