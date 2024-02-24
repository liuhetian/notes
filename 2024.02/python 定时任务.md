# Python 定时任务

写接口的时候遇到了定时更新环境变量的问题，以此为契机查一下Python中定时任务除了死循环还有什么好的办法

```python
from apscheduler.schedulers.blocking import BlockingScheduler
import datetime

# 这是我们想要定时执行的函数
def my_job():
    print(f"Job executed at {datetime.datetime.now()}")

# 创建一个调度器
scheduler = BlockingScheduler()

# 添加一个作业，每10秒执行一次
scheduler.add_job(my_job, 'interval', seconds=10)

try:
    # 开始运行调度器
    scheduler.start()
except (KeyboardInterrupt, SystemExit):
    # 当程序退出时停止调度器
    scheduler.shutdown()

```
