# Python 多线程
python中的经典问题

## 最简单实现
最简单的办法是使用`Threading`库
```python
import threading
import time 

def f():
    while True:
        print('test')
        time.sleep(5)
        
t = threading.Thread(target=f)
t.start()
```

## 守护线程
上面的案例中脚本会一直运行，原因是Python的线程默认是非守护的，也就是子线程不是为了在服务主进程，
虽然主进程里的代码执行光了，仍然要等到子线程执行结束脚本才会退出。
在有的情况下，例如后台一个线程每隔一定时间更新环境变量，这种线程就是为了主进程服务的，如果主进程退出，
这样的线程也就可以立刻退出，所以需要设置为守护线程

```python
import threading
import time 

def f():
    while True:
        print('test')
        time.sleep(5)
        
t = threading.Thread(target=f)
t.daemon = True
t.start()

time.sleep(20)  # 这20s就是留给子线程可以执行的时间

```

```
(base) ubuntu# python test.py 
test
test
test
test
```

### 手动实现非守护线程
如果想让守护线程手动变为非守护线程（正常没有这个需求，线程默认就是非守护的）
可以使用join方法

```python
import threading
import time 

def f():
    while True:
        print('test')
        time.sleep(5)
        
t = threading.Thread(target=f)
t.daemon = True
t.start()

time.sleep(1)  # 这1s就是留给子线程可以执行的时间
t.join()  # 等子线程执行完
```

这样就又变回永不停歇地print了

## Threading

## 线程池
