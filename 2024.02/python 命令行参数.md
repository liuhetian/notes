# Python 命令行参数

运行线上脚本总有很多东西需要设置，例如有的脚本手动刷的时候要
1. 控制刷数据的时间范围
2. 取数方式（从中间表还是从原始表取）
3. 内网还是外网环境
4. 线程数量
5. etc...

很多配置脚本刚写好还记得，但是久了就真的一点印象没有，所以设想2种方式来运行脚本

1. python main.py a b c （默认的运行方式）
2. python main.py （有可能会临时手动运行该脚本，传入参数就可能出错，所以希望脚本用pydantic验证命令行参数是否符合要求，如果不符合要求就自动启用prompt_toolkit来收集参数）
3. 

## prompt_toolkit

找了两个月最终确定完美符合需求的方法，

不符合要求就无法enter

稍微要注意一下ValidationError的用法，有message和cursor_position两个参数，


```python
from prompt_toolkit.validation import Validator, ValidationError
from prompt_toolkit import prompt


class StrValidator(Validator):
    def validate(self, document):
        text = document.text

        if len(text) < 3:
            raise ValidationError(message='太短了', cursor_position=len(text))
        
        if not text.endswith('test'):
            raise ValidationError(message='结尾不是test', cursor_position=len(text))
            
s = prompt('Give a str: ', validator=StrValidator())
print('You said: %s' % s)
```

## 实践方案

[pydantic](https://github.com/pydantic/pydantic)(18k)

[prompt_toolkit validator](https://python-prompt-toolkit.readthedocs.io/en/stable/pages/reference.html#prompt_toolkit.validation.Validator)
文档里说:

```
abstract validate(document: Document) → None
Validate the input. If invalid, this should raise a ValidationError.
```
所以是根据是否报错来看是否通过，然后通过return text，就可以


```python
# b.py
import argparse
from prompt_toolkit.validation import Validator, ValidationError
from prompt_toolkit import prompt
from types import SimpleNamespace

def validate1(text):
    if len(text) < 4:
        raise ValidationError(message='太短了', cursor_position=len(text))
    if not text.endswith('test'):
        raise ValidationError(message='结尾不是test', cursor_position=len(text))
    return text
     
class Validator1(Validator):
    def validate(self, document):
        text = document.text
        validate1(text)
        
def validate2(text):
    # Validator.from_callable这种校验方式需要返回True或者False
    # 如果需要各种详细信息，需要
    # if not <condition>:
    #     raise ValidationError(message='原因', cursor_position=len(text))
    # if not <condition>:
    #     raise ...
    return text.isdigit() and (0 <= eval(text) < 15)

validator2 = Validator.from_callable(
    validate2,
    error_message='不符合规范',
    move_cursor_to_end=True
)

def get_arg():
    args = None
    try:
        parser = argparse.ArgumentParser(description="这是一个示例脚本，说明如何使用argparse处理命令行参数。")
        parser.add_argument("param1", type=validate1, help="第二个参数的说明")
        parser.add_argument("param2", type=lambda x: x if validate2(x) else 1/0, help="第一个参数的说明")  # 不常用
        args = parser.parse_args()
    except SystemExit as e:
        if e.code == 0:
            raise  # -h参数结束程序
    except ValidationError as e:
        print(e.message)
    except:
        print('参数填写不正确，请重新填写：')
    if args is None:
        arg1 = prompt("param1: ", validator=Validator1())
        arg2 = prompt("param2(整数): ", default="1.0", validator=validator2)
        args = SimpleNamespace(param1=arg1, param2=arg2)

    print(f"参数1: {args.param1}, 参数2: {args.param2}")
    return args
```

```python
# a.py
from b import get_arg
if __name__ == '__main__':
    a = get_arg()
    print(a)
```

```bash
python a.py -h  # 查看参数
python a.py test 1  # 正确运行
python a.py  # 直接开始使用toolkit获取参数
python a.py t a  # 参数不正确，使用toolkit获取参数
```

## 其他办法

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
def main(count, name):
    """这个脚本的用途是xxx"""
    for _ in range(count):
        print(f"Hello, {name}!")

if __name__ == '__main__':
    print(__doc__)
    main()
```

## typer

在click之上进行一层封装





