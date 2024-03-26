# Python 命令行参数

运行线上脚本总有很多东西需要设置，例如有的脚本手动刷的时候要
1. 控制刷数据的时间范围
2. 取数方式（从中间表还是从原始表取）
3. 内网还是外网环境
4. 线程数量
5. etc...

很多配置脚本刚写好还记得，但是久了就真的一点印象没有

## 原生
感觉就是调用了$0, $1, $2, $3？

```python
import sys
print(sys.argv)
```

```bash
python test.py a b c
['test.py', 'a', 'b', 'c']
```

## argparse

功能强大但是用起来很麻烦，不够友好

```python
# 先创建
parser = argparse.ArgumentParser(
  prog='ProgramName',
  description='What the program does',
  epilog='Text at the bottom of help'
)

parser.add_argument('filename')           # positional argument
parser.add_argument('-c', '--count')      # option that takes a value
parser.add_argument('-v', '--verbose', action='store_true')  # on/off flag
```

## docopt
理念很有意思，但是很多年没维护

## click
非常友好，可以直接运行然后让用户输入和选择，但是从外部不能获取到这些参数，只能在它的函数里运行主函数

```python
import click

@click.command()
@click.option('--int_', prompt='请输入整数', type=int, help='')
@click.option('--str_', prompt='请输入字符串', type=str, help='')
@click.option('--bool_', prompt='请输入是或者否', is_flag=True, help='')
def get_params(int_, str_, bool_):
    print(int_, str_, bool_)  # 从外部无法获取这些参数

```

以后线上脚本就用这个模版，click库作为程序入口

```python
"""
这个脚本的用途是xxx
线上应该使用xxx命令运行
可以使用python main.py --help命令查看更多选项
"""

import click

@click.command()
@click.option("--count", default=1, help="重复多少次")
@click.option("--name", prompt="Your name", help="打招呼的名字是什么")
def hello(count, name):
    """这个脚本的用途是xxx"""
    for _ in range(count):
        print(f"Hello, {name}!")

if __name__ == '__main__':
    print(__doc__)
    hello()
```

## typer

在click之上进行一层封装



