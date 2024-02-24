# Fastapi

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

