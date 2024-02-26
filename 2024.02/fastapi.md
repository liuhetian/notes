# Fastapi
记录一些遇到的问题

## 运行fastapi
使用uvicorn或者gunicorn运行

使用`uvicorn main:app`运行`main.py`里的`app`，感觉上是在uvicorn的脚本里`from main import app`，
python的机制是import的话会把代码全部运行一遍，但是app在的变量空间啥的还是有点疑惑，如下：


```python
import asyncio
from fastapi import FastAPI
app = FastAPI()
print('这行代码会正常运行吗？')  # 会print出来
@app.get("/")
async def f():
    return {"Hello": "World"}
```

```bash
uvicorn main:app
```

## 环境变量与更新

需要用到一个配置表DataFrame，需要从Google Sheet里面读取，然后每个小时定时更新
有`app.state`和全局变量两种形式，似乎差不多

## 简单实现

最简单的实现方式就是使用上面的代码，但是把print改成增加一个死循环线程，

但是如果把`print`换成一个运行的线程感觉就会很神奇，我们写的脚本相当于一个库，然后uvicorn的脚本调用后执行，
现在我们的库里有一个子线程，相当于一个库(`main.py`)不是静态的了

```python
from fastapi import FastAPI
app = FastAPI()

#--------------
import threading, time
from datetime import datetime

# 为线程定义一个函数
def f():
    global t
    for i in range(20):
        time.sleep(5)
        print(i)
        t = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

thread = threading.Thread(target=f)
thread.daemon = True
thread.start()
#--------------

@app.get("/")
async def f():
    return {"data": "Hello World!", 'time': t}
```

```bash
(base) root@VM-4-2-ubuntu:/home/ubuntu/main/2.24# uvicorn test2:app --host=0.0.0.0 --port=8000
INFO:     Started server process [3810872]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
0
1
2
...
```

```
# 127.0.0.1：8000
{"data":"Hello World!","time":"2024-02-24 10:15:10"}
{"data":"Hello World!","time":"2024-02-24 10:15:15"} 
{"data":"Hello World!","time":"2024-02-24 10:15:20"}
...
{"data":"Hello World!","time":"2024-02-24 10:15:50"}
```

所以的确可以通过这种方式来定时更新环境变量，感觉好神奇

可以用更优雅一点点的办法写，但仍然很简单。
threading.Timer创建的是普通线程（非守护），可以设置`thread.daemon = True`

```python
import threading

def update():
    print('更新代码')
    threading.Timer(5, update).start()

update()
print('主脚本代码执行到这里')
```

## 使用定时任务库
上述的简单方法通过死循环的方式实现了全局变量的更新，但是代码呈现比较混乱，所以需要库来实现

### 使用schedule
```bash
pip install schedule
```
```python
import time, threading, schedule
from datetime import datetime

def job():
    global t
    t = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("test")

def update():
    schedule.every(5).seconds.do(job)
    while True:
        schedule.run_pending()
        time.sleep(1)

thread = threading.Thread(target=update)
thread.daemon = True
thread.start()

t = '初始值'
while True:
    print(t)
    time.sleep(5)
```

写完发现纯纯多此一举啊

### 使用apscheduler
apscheduler有背景任务很适合这个场景

```python
import time
from datetime import datetime
from apscheduler.schedulers.background import BackgroundScheduler

scheduler = BackgroundScheduler()
def update():
    global t
    t = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
@scheduler.scheduled_job("interval", seconds=5)
def scheduled_job():
    update()
scheduler.start()

t = '初始值'
while True:
    print(t)
    time.sleep(5)
```

### 使用fastapi生命周期

```python
from fastapi import FastAPI
from datetime import datetime
from contextlib import asynccontextmanager
import asyncio

def update():
    global t
    t = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("配置表更新完毕...")

async def update_config():
    while True:
        await asyncio.sleep(5)
        update()
        
@asynccontextmanager
async def lifespan(app: FastAPI):
    update()
    asyncio.create_task(update_config())
    print('初始化成功')
    yield
    print('fastapi退出')
    
app = FastAPI(lifespan=lifespan)

@app.get("/")
async def f():
    return t
```

### 使用fastapi.BackgroundTasks

```python
#app = FastAPI()
background_tasks = BackgroundTasks()


@asynccontextmanager
async def lifespan(app: FastAPI):
    update()
    background_tasks.add_task(update_config)
```

启动和关闭时要做一些事情，例如创建/关闭httpx客户端
[文档-寿命事件](https://fastapi.tiangolo.com/advanced/events/)
