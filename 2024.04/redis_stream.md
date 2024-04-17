# Redis Stream

操作redis里的stream结构

## 放入

放入比较简单

```python
stream_name = 'task_stream'
task_data = {'task_type': 'email', 'email_address': 'user@example.com'}
task_id = r.xadd(stream_name, task_data)
print(task_id)

1713336700247-0

# 测量长度 理论上非常非常大
r.xlen('task_stream')
response = r.xtrim(stream_name, maxlen=3, approximate=False)  # response是修剪了多少
# flushdb()会把stream也删掉
```

## 取出

取出是个非常复杂的事情

### 简单取出

```python
messages = r.xrange(stream_name)
```

### 按照时间取出
以最近1小时为例：
```python
from datetime import datetime, timedelta
import time

stream_name = 'p2_gpt_shop_bargain_lht'
hour_ago = datetime.now() - timedelta(hours=1)
start_id = str(int(time.mktime(hour_ago.timetuple())*1000)) + '-0'
tasks = r.xrange(stream_name, min=start_id, max='+')
```

### 堵塞读取

```python
# 阻塞并等待新消息
# block=None表示永久阻塞，直到有新消息到来。$代表从最新的消息开始读取
while True:
    results = r.xread({stream_name: '$'}, block=None)
    for stream, messages in results:
        for message_id, message in messages:
            print(f"Received message ID: {message_id}, Message: {message}")
```

### 使用任务组有记忆地读取

场景就很复杂，例如我有一个任务，由有很多台服务器的微服务/多个线程来处理，要避免多个线程拿到同一个任务

要注意的是为了避免污染stream，要把这些微任务打包成一个组，方便统一管理

读取的时候字典`{stream_name: '>'}` 中 `>` 这个特殊的ID表示该消费者（consumer）希望从该流中读取尚未被该消费者组中的任何消费者读取的消息。换句话说，`>` 指的是该流中所有新消息的开始位置，仅对当前消费者组中尚未被任何消费者确认的消息。

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)
stream_name = 'mystream'
group_name = 'mygroup'

# 尝试创建 Consumer Group，忽略已存在的错误
try:
    r.xgroup_create(stream_name, group_name, mkstream=True)
except redis.exceptions.ResponseError as e:
    if "BUSYGROUP Consumer Group name already exists" not in str(e):
        raise

consumer_name = 'worker1'
while True:
    results = r.xreadgroup(group_name, consumer_name, {stream_name: '>'}, block=None)
    for stream, messages in results:
        for message_id, message in messages:
            try:
                print(f"Processing message ID: {message_id}, Message: {message}")
                # 处理消息逻辑...
                r.xack(stream_name, group_name, message_id)
            except Exception as e:
                print(f"Failed to process message ID: {message_id}, Error: {e}")
                # 可以在这里处理错误，例如重试或记录错误

```

从失败中恢复并处理所有消息：要注意的是可能有任务执行中出错，导致没有被ack，对于这样的任务即便脚本重新运行也不会再次读取，需要采取额外的步骤（如 XPENDING 和 XCLAIM 或从特定ID重新读取）来处理这类“挂起”的消息。
