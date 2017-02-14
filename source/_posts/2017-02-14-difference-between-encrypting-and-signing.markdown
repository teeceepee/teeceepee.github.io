---
layout: post
title: "加密与签名的区别"
date: 2017-02-14 22:48
comments: true
categories: 
---

最近要在 Redis 里放一点敏感数据，于是就在 Rails 里找实现加密的东西。Rails 有 `cookies.encrypted` 和 `cookies.signed` 两个方法，区别一直没搞明白，这次研究了一下，终于明白了。

## 加密

加密的目的是保证数据只能通过秘钥才能查看，没有秘钥看不到任何数据。

## 签名

签名的目的是保证数据不会被篡改或伪造，但是可以在没有秘钥的情况下查看原数据！

## 示例

ActiveSupport 中有加密和签名的功能，对应的类是 ActiveSupport::MessageEncryptor 和 ActiveSupport::MessageVerifier。下面的例子展示了它们的用法以及区别。

``` ruby
secret = 'secret' * 10
encryptor = ActiveSupport::MessageEncryptor.new(secret)

encrypted_data = encryptor.encrypt_and_sign('You cannot see me')

# with secret
encryptor.decrypt_and_verify(encrypted_data)
# => "You cannot see me"
# You can do nothing without secret
```

``` ruby
secret = 'secret' * 10
verifier = ActiveSupport::MessageVerifier.new(secret)
signed_data = verifier.generate('You cannot see me')

# with secret
verifier.verify(signed_data)
# => "You cannot see me"

# without secret !!!
encoded = signed_data.split('--').first
Marshal.load(Base64.strict_decode64(encoded))
# => "You cannot see me"
```

值得一提的是，ActiveSupport::MessageEncryptor 除了加密，也提供了签名的功能，从方法名中可以看出这一点。


## 参考

[Reading Rails - How Does MessageVerifier Work?](http://www.monkeyandcrow.com/blog/reading_rails_how_does_message_verifier_work/)

[Reading Rails - How Does MessageEncryptor Work?](http://www.monkeyandcrow.com/blog/reading_rails_how_does_message_encryptor_work/)
