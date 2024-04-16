# Pathlib

``` python
from pathlib import Path

a = Path('bar.txt')
# PosixPath('bar.txt')

a.parent
# PosixPath('.')

a.parent.parent
# 还是PosixPath('.')不变，感觉有点问题

b = a.resolve()  # .absolute() 相比，对于符号链接的解析有点不一样
b.parent
# PosixPath('/home/ubuntu/main/4.1')  # 这就对了


b.stem, b.suffix
# ('bar', '.txt')

Path('a.b.c').stem
# 'a.b'
```
