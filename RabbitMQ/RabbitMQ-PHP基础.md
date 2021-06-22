

Base

`send.php`

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建连接
$connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
//创建信道
$channel = $connection->channel();

// 创建队列
$channel->queue_declare('hello', false, false, false, false);

// 发送消息
$msg = new AMQPMessage("Hello World!");
$channel->basic_publish($msg, '', 'hello');
echo " [x] Sent 'Hello World!'\n";

// 关闭信道
$channel->close();
// 关闭连接
$connection->close();
```

`receive.php`

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('127.0.0.1', 5672, 'doubi', '123456');
$channel = $connection->channel();

$channel->queue_declare('hello', false, false, false, false);

echo " [*] Waiting for messages. To exit press CTRL+C\n";

$callback = function ($msg) {
  echo ' [x] Received ', $msg->body, "\n";
};

$channel->basic_consume('hello', '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}
```



Work queues



Publish/Subscribe



Routing



Topics



RPC



Publisher Confirms

