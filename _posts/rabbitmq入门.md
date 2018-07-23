# RabbitMQ

`The main idea behind Work Queues (aka: Task Queues) is to avoid doing a resource-intensive task immediately and having to wait for it to complete.`

译:工作队列(任务队列)背后的主要思想是为了避免立即做一个资源密集型任务,不得不等待它完成

## 环境

RabbitMQ主机地址：`192.168.182.128`，端口：`5672`

安装依赖库 `npm install amqplib`


## 生产者消费者hello world版

### 发送端：
```
#!/usr/bin/env node

const amqp = require("amqplib/callback_api")

amqp.connect('amqp://192.168.182.128', function(err, conn) {
  conn.createChannel(function(err, ch) {
    var q = 'hello';

    ch.assertQueue(q, {durable: false});
    // Note: on Node 6 Buffer.from(msg) should be used
    ch.sendToQueue(q, new Buffer('Hello World!'));
    console.log(" [x] Sent 'Hello World!'");
    setTimeout(function() { conn.close(); process.exit(0) }, 500);
  });
});

```

### 接收端：
```
#!/usr/bin/env node

var amqp = require('amqplib/callback_api');

amqp.connect('amqp://192.168.182.128', function(err, conn) {
  conn.createChannel(function(err, ch) {
    var q = 'hello';

    ch.assertQueue(q, {durable: false});
    ch.consume(q,msg=>{
      console.log("recive:",msg.content.toString())
    });
  });
})

```
生产者发送消息需要指定channel后再把消息发到到队列。


## 多消费者

### 生产者
```
#!/usr/bin/env node

const amqp = require("amqplib/callback_api")

amqp.connect('amqp://192.168.182.128', function(err, conn) {
  conn.createChannel(function(err, ch) {
    var q = 'task_queue'
    var msg = process.argv.slice(2).join(' ')||'hello wrold'

    ch.assertQueue(q, {durable: false});
   
    ch.sendToQueue(q, new Buffer(msg),{persistent:true});
    console.log(" [x] Sent '%s'",msg);
    setTimeout(function() { conn.close(); process.exit(0) }, 500);
  });
});

```


### 消费者
```
#!/usr/bin/env node

var amqp = require('amqplib/callback_api');

amqp.connect('amqp://192.168.182.128', function(err, conn) {
  conn.createChannel(function(err, ch) {
    var q = 'task_queue';

    ch.assertQueue(q, {durable: false});
    ch.consume(q,msg=>{
      var secs = msg.content.toString().split('.').length - 1;
      console.log(" [x] Received %s", msg.content.toString());
      setTimeout(function() {
        console.log(" [x] Done");
      }, secs * 1000);
    }, {noAck: false});
  });
})
```
`{durable: false}`这个设置是否持久化，为`false`：RabbitMQ关闭了，消息就没有了。

`{noAck: false}`如果这个设置成`true`，消费者不会发送消息确认，即消费者挂了分配给这个消费者的消息也就没了。设置成`false`，消费者挂了，RabbitMQ会把该消费者的消息分配给其他在线的消费者。