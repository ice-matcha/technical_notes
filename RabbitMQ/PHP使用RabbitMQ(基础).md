PHP使用RabbitMQ(基础)

介绍



基础入门

```php
/**
 * 基础入门
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function senderAccidence()
{
    //创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    //开启信道
    $channel = $connection->channel();

    /**
     * 声明一个队列,以下为创建队列参数解析
     * queue：队列的名称
     * passive：
     * durable：是否持久化,true表示持久化
     * exclusive：是否排他。true表示排他
     * auto_delete：是否自动删除。true表示自动删除
     * nowait：
     * arguments：设置队列的其他一些参数
     * ticket：
     */
    $channel->queue_declare('hello', false, false, false, false);

    // 消息推送
    $msg = new AMQPMessage("Hello ice-matcha");
    $channel->basic_publish($msg, '', 'hello');

    echo " [x] Sent Hello ice-matcha \n";

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```

接收：

```php
/**
 * 基础入门
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function receiverAccidence()
{
    // 创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    // 创建信道
    $channel = $connection->channel();
    // 创建队列
    $channel->queue_declare('hello', false, false, false, false);

    echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

    // 获取队列消息
    $channel->basic_consume('hello', '', false, true, false, false, function($msg){
        echo " [x] Received ", $msg->body, "\n";
    });

    while(count($channel->callbacks)) {
       $channel->wait();
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```



工作队列

发送：

```php
/**
 * 工作队列
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return void
 */
public function senderWorkQueue()
{
    //创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    //开启信道
    $channel = $connection->channel();

    $channel->queue_declare('task_queue', false, true, false, false);

    $i = 1;
    while ($i <= 20) {
        $data = "Hello World!{$i}";
        $msg = new AMQPMessage($data, [
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        $channel->basic_publish($msg, '', 'task_queue');

        echo " [x] Sent ", $data, "\n";
        $i++;
        sleep(1);
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```

接收：

```php
/**
 * 工作队列
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function receiverWorkQueue()
{
    // 创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    // 创建信道
    $channel = $connection->channel();

    $channel->queue_declare('task_queue', false, true, false, false);

    echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

    // 负载均衡
    $channel->basic_qos(null, 1, null);
    $channel->basic_consume('task_queue', '', false, false, false, false, function($msg) {
        echo " [x] Received ", $msg->body, "\n";
        sleep(2);
        echo " [x] Done", "\n";
        $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
    });

    while(count($channel->callbacks)) {
        $channel->wait();
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```



发布/订阅

发送：

```php
/**
 * 发布订阅
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function senderPublishSubscribe()
{
    //创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    //开启信道
    $channel = $connection->channel();

    /**
     * 创建交换器,以下为创建交换器参数解析
     * exchange：交换器的名称
     * type：交换器的类型,常见的如fanout、direct、topic
     * passive：
     * durable：是否持久化,true表示持久化
     * auto_delete：是否自动删除,true表示自动删除
     * internal：
     * nowait：
     * arguments：其他一些结构化参数
     * ticket：
     */
    $channel->exchange_declare('logs', 'fanout', false, false, false);

    $data = "info: Hello World!";
    $msg = new AMQPMessage($data);

    $channel->basic_publish($msg, 'logs');

    echo " [x] Sent ", $data, "\n";

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```

接收：

```php
/**
 * 发布订阅
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function receiverPublishSubscribe()
{
    // 创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    // 创建信道
    $channel = $connection->channel();
    // 创建交换机
    $channel->exchange_declare('logs', 'fanout', false, false, false);

    // 获取队列名称
    list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

    // 绑定队列
    $channel->queue_bind($queue_name, 'logs');

    echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

    $channel->basic_consume($queue_name, '', false, true, false, false, function($msg) {
        echo ' [x] ', $msg->body, "\n";
    });

    while(count($channel->callbacks)) {
        $channel->wait();
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```



路由

发送：

```php
/**
 * 路由
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function senderRouter()
{
    //创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    //开启信道
    $channel = $connection->channel();

    $channel->exchange_declare('direct_logs', 'direct', false, false, false);

    // $severity = 'info';
    $severity = 'debug';

    $data = "Hello World!";

    $msg = new AMQPMessage($data);

    $channel->basic_publish($msg, 'direct_logs', $severity);

    echo " [x] Sent ",$severity,':',$data," \n";

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```

接收：

```php
/**
 * 路由
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function receiverRouter()
{
    // 创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    // 创建信道
    $channel = $connection->channel();

    $channel->exchange_declare('direct_logs', 'direct', false, false, false);

    list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

    // 队列与路由绑定
    $severities = ['info','debug'];
    foreach ($severities as $severity) {
        $channel->queue_bind($queue_name, 'direct_logs', $severity);
    }

    echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

    $channel->basic_consume($queue_name, '', false, true, false, false, function($msg){
        echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
    });

    while(count($channel->callbacks)) {
        $channel->wait();
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```



话题交换

发送：

```php
/**
 * 话题交换
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function senderTopic()
{
    //创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    //开启信道
    $channel = $connection->channel();

    $channel->exchange_declare('topic_logs', 'topic', false, false, false);

    // $routing_key = 'anonymous.info';
    $routing_key = 'anonymous.debug';
    $data = "Hello World!";

    $msg = new AMQPMessage($data);

    $channel->basic_publish($msg, 'topic_logs', $routing_key);

    echo " [x] Sent ",$routing_key,':',$data," \n";

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```

接收：

```php
/**
 * 话题交换
 * Created by sublime text 3
 * @Author   ice-matcha
 * @DateTime 2020-12-25
 * @return   void
 */
public function receiverTopic()
{
    // 创建连接
    $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
    // 创建信道
    $channel = $connection->channel();

    $channel->exchange_declare('topic_logs', 'topic', false, false, false);

    list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

    $binding_keys = ['anonymous.info', 'anonymous.debug'];

    foreach($binding_keys as $binding_key) {
        $channel->queue_bind($queue_name, 'topic_logs', $binding_key);
    }

    echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

    $channel->basic_consume($queue_name, '', false, true, false, false, function($msg){
        echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
    });

    while(count($channel->callbacks)) {
        $channel->wait();
    }

    // 关闭信道
    $channel->close();
    // 关闭连接
    $connection->close();
}
```



远程调用(RPC)

