---
title: "关于logstash-output-mongodb的一点质疑"
lead: "02 未划分"
date: 2023-04-22T12:52:56+08:00
lastmod: 2023-04-22T12:52:56+08:00
draft: false
images: []
menu:
  architecture:
    parent: "devops"
weight: 4000
toc: true
---

# 背景
使用的Logstash版本是7.4.2，使用的logstash-output-mongodb版本是3.1.6。基本配置如下：
```
input {
  beats {
    port => 5044
  }
}

filter {
  mutate {
    remove_field => ["log_type"]
    remove_field => ["offset"]
    remove_field => ["input"]
    remove_field => ["log"]
    remove_field => ["input"]
    remove_field => ["log"]
    remove_field => ["ecs"]
    remove_field => ["@version"]
    remove_field => ["@timestamp"]
    remove_field => ["host"]
    remove_field => ["agent"]
    remove_field => ["tags"]
  }
  json {
     source => "message"
     remove_field => ["message"]
  }
  geoip {
    source => "remote_addr"
    target => "addr_info"
  }
  mutate {
    remove_field => ["tags"]
  }
}

output {
  stdout {}
    mongodb {
      collection => "XXXX"
      database => "XXXX"
      uri => "mongodb://XXX:XXX@XXX:27017"
    }
}
```

# 问题
- ## logstash的问题
1. 时间戳logstash自动改，而且还不带时区。
最主要的问题是大家的处理方法都是给时间加上8个小时来处理时间。查找Logstash的时区设置很难找到相关资料，并且所有的参数设置后都无法生效。

2. ClassCastException在使用output-mongodb时发生，使用其他输出时就不发生。
```
:exception=>java.lang.ClassCastException: org.jruby.RubyNil cannot be cast to org.jruby.RubyFixnum (org.jruby.RubyNil and org.jruby.RubyFixnum are in unnamed module of loader 'app'), 
:backtrace=>["org.jruby.runtime.invokedynamic.MathLinker.fixnum_op_equal(MathLinker.java:237)", 
"java.lang.invoke.MethodHandle.invokeWithArguments(MethodHandle.java:627)", 
"org.jruby.runtime.invokedynamic.MathLinker.fixnumOperator(MathLinker.java:171)", 
"usr.share.logstash.vendor.bundle.jruby.$2_dot_5_dot_0.gems.mongo_minus_2_dot_11_dot_0.lib.mongo.server_selector.selectable.RUBY$method$initialize$0(/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server_selector/selectable.rb:46)", 
"org.jruby.internal.runtime.methods.CompiledIRMethod.call(CompiledIRMethod.java:91)", 
"org.jruby.internal.runtime.methods.MixedModeIRMethod.call(MixedModeIRMethod.java:90)", 
"org.jruby.runtime.callsite.CachingCallSite.cacheAndCall(CachingCallSite.java:332)", 
"org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:86)", 
"org.jruby.RubyClass.newInstance(RubyClass.java:915)", 
"org.jruby.RubyClass$INVOKER$i$newInstance.call(RubyClass$INVOKER$i$newInstance.gen)", 
"org.jruby.ir.targets.InvokeSite.invoke(InvokeSite.java:183)", 
"usr.share.logstash.vendor.bundle.jruby.$2_dot_5_dot_0.gems.mongo_minus_2_dot_11_dot_0.lib.mongo.server_selector.RUBY$method$get$0(/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server_selector.rb:75)", 
"org.jruby.internal.runtime.methods.CompiledIRMethod.call(CompiledIRMethod.java:91)", 
"org.jruby.internal.runtime.methods.MixedModeIRMethod.call(MixedModeIRMethod.java:90)", 
"org.jruby.ir.targets.InvokeSite.invoke(InvokeSite.java:183)", 
"usr.share.logstash.vendor.bundle.jruby.$2_dot_5_dot_0.gems.mongo_minus_2_dot_11_dot_0.lib.mongo.server_selector.RUBY$method$primary$0(/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server_selector.rb:85)", 
"usr.share.logstash.vendor.bundle.jruby.$2_dot_5_dot_0.gems.mongo_minus_2_dot_11_dot_0.lib.mongo.server_selector.RUBY$method$primary$0$__VARARGS__(/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server_selector.rb)", 
"org.jruby.internal.runtime.methods.CompiledIRMethod.call(CompiledIRMethod.java:91)", 
"org.jruby.internal.runtime.methods.MixedModeIRMethod.call(MixedModeIRMethod.java:90)", 
"org.jruby.ir.targets.InvokeSite.invoke(InvokeSite.java:183)", 
"usr.share.logstash.vendor.bundle.jruby.$2_dot_5_dot_0.gems.mongo_minus_2_dot_11_dot_0.lib.mongo.retryable.RUBY$method$legacy_write_with_retry$0(/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/retryable.rb:280)", 
"org.jruby.internal.runtime.methods.CompiledIRMethod.call(CompiledIRMethod.java:91)", 
"org.jruby.internal.runtime.methods.MixedModeIRMethod.call(MixedModeIRMethod.java:90)", 
"org.jruby.ir.targets.InvokeSite.invoke(InvokeSite.java:183)", 
```

- ## logstash-output-mongodb问题

1. 一次异常一直阻塞的问题。这个里面明显看到接受到异常之后重试机制，但是重试无限制。导致整个线程等的这个消息
```
    rescue => e
      if e.message =~ /^E11000/
        # On a duplicate key error, skip the insert.
        # We could check if the duplicate key err is the _id key
        # and generate a new primary key.
        # If the duplicate key error is on another field, we have no way
        # to fix the issue.
        @logger.warn("Skipping insert because of a duplicate key error", :event => event, :exception => e)
      else
        @logger.warn("Failed to send event to MongoDB, retrying in #{@retry_delay.to_s} seconds", :event => event, :exception => e)
        sleep(@retry_delay)
        retry
      end
```
2. 不输出堆栈，不输出事件
代码同上，在pipeline进行处理过程中不打印堆栈、不打印时间内容。只知道发生问题了，没有办法定位是那个事件造成的，不知道具体是哪行代码产生了问题。

3. 数字格式问题
```
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/logstash-output-mongodb-3.1.6/lib/logstash/outputs/bson/big_decimal.rb:-1:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:58:in `block in to_bson'", 
"org/jruby/RubyHash.java:1417:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:46:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:332:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:293:in `block in serialize'", 
"org/jruby/RubyArray.java:1800:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:292:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:181:in `block in serialize'", 
"org/jruby/RubyArray.java:1800:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:174:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/message.rb:237:in `block in serialize_fields'", 
```
4. 错误的参数个数
```
:exception=>#<ArgumentError: wrong number of arguments (given 2, expected 0..1)>, 
:backtrace=>[
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/logstash-output-mongodb-3.1.6/lib/logstash/outputs/bson/big_decimal.rb:-1:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:58:in `block in to_bson'", 
"org/jruby/RubyHash.java:1417:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:46:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:58:in `block in to_bson'", 
"org/jruby/RubyHash.java:1417:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:46:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:58:in `block in to_bson'", 
"org/jruby/RubyHash.java:1417:in `each'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:46:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/array.rb:52:in `block in to_bson'", "org/jruby/RubyArray.java:1800:in `each'", 
"org/jruby/RubyEnumerable.java:1237:in `each_with_index'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/array.rb:49:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:58:in `block in to_bson'", 
"org/jruby/RubyHash.java:1417:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/bson-4.6.0-java/lib/bson/hash.rb:46:in `to_bson'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:332:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:293:in `block in serialize'", 
"org/jruby/RubyArray.java:1800:in `each'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:292:in `serialize'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:181:in `block in serialize'", "org/jruby/RubyArray.java:1800:in `each'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/serializers.rb:174:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/message.rb:237:in `block in serialize_fields'", 
"org/jruby/RubyArray.java:1800:in `each'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/message.rb:225:in `serialize_fields'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/message.rb:126:in `serialize'", "/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/protocol/msg.rb:126:in `serialize'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/sessions_supported.rb:166:in `block in build_message'", "org/jruby/RubyKernel.java:1885:in `tap'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/sessions_supported.rb:161:in `build_message'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:56:in `block in dispatch_message'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server/connection_pool.rb:557:in `with_connection'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/server.rb:410:in `with_connection'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:55:in `dispatch_message'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/insert/op_msg.rb:35:in `get_result'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:29:in `block in do_execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/response_handling.rb:87:in `add_server_diagnostics'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:28:in `block in do_execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/response_handling.rb:43:in `add_error_labels'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:27:in `block in do_execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/response_handling.rb:73:in `unpin_maybe'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable.rb:26:in `do_execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/executable_no_validate.rb:25:in `execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/operation/shared/write.rb:38:in `execute'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/collection.rb:538:in `block in insert_one'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/retryable.rb:281:in `legacy_write_with_retry'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/retryable.rb:197:in `write_with_retry'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/collection.rb:537:in `block in insert_one'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/cluster.rb:799:in `with_session'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/client.rb:837:in `with_session'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/mongo-2.11.0/lib/mongo/collection.rb:535:in `insert_one'", 
"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/logstash-output-mongodb-3.1.6/lib/logstash/outputs/mongodb.rb:111:in `receive'", 
"/usr/share/logstash/logstash-core/lib/logstash/outputs/base.rb:89:in `block in multi_receive'", "org/jruby/RubyArray.java:1800:in `each'", 
"/usr/share/logstash/logstash-core/lib/logstash/outputs/base.rb:89:in `multi_receive'", "org/logstash/config/ir/compiler/OutputStrategyExt.java:118:in `multi_receive'", 
"org/logstash/config/ir/compiler/AbstractOutputDelegatorExt.java:101:in `multi_receive'", 
"/usr/share/logstash/logstash-core/lib/logstash/java_pipeline.rb:243:in `block in start_workers'"
```

# 总结
不知道logstash-output-mongodb是不是还在实验阶段，没有正式发布。
