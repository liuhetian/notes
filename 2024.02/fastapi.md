# Fastapi
记录一些遇到的问题

## uvicorn如何运行代码

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

上面的例子比较简单还可以理解，但是如果把`print`换成一个运行的线程感觉就会很神奇，相当于一个库(`main.py`)不是静态的了


```python
from fastapi import FastAPI
app = FastAPI()

#print('这行代码会正常运行吗？')
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
thread.start()

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
```

```
# 127.0.0.1：8000
{"data":"Hello World!","time":"2024-02-24 10:15:10"}
{"data":"Hello World!","time":"2024-02-24 10:15:15"} 
{"data":"Hello World!","time":"2024-02-24 10:15:20"}
...
{"data":"Hello World!","time":"2024-02-24 10:15:50"}
```

所以的确可以通过这种方式来定时更新环境变量，感觉好神奇啊

(发现个问题，子线程执行结束之前，ctrl+C关闭不了进程？)

## 环境变量与更新
需要用到一个配置表DataFrame，需要从Google Sheet里面读取，然后每个小时定时更新
有`app.state`和全局变量两种形式，似乎差不多
（还没研究明白）

## 寿命事件
启动和关闭时要做一些事情，例如创建/关闭httpx客户端
[文档-寿命事件](https://fastapi.tiangolo.com/advanced/events/)
