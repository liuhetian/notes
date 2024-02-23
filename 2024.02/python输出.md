# python输出

有些时候想输出更好看的内容，例如颜色啥的，
## print
从print开始，常用的是控制分隔符sep和结束的end，在流式传输的时候用到过flush参数，如果是True就不会缓存直接输出，

另外在服务器上跑脚本发现所有输出内容都等代码运行的差不多了再一下子print出来，怀疑也和这个flush参数有关。
另外没有注意到可以直接print到文件，感觉很神奇，不知道线程安全性怎么样。

```
print(value,...,sep='',end='\n',file=sys.stdout,flush=False)
```

## rich

## ...
