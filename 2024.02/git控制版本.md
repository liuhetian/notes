# 控制版本

2024到来，要优化代码，先从版本控制开始，现在的控制方法很呆：
```
lht_p2_bargain/
├── 2.x
│   ├── 2.0
│   ├── 2.1
│   └── env
├── 3.x
│   ├── 3.0
│   ├── 3.1
│   └── env
```

## 修改名称
```bash
git config --global user.name "Liu Hetian"
git config --global user.email "liuhetian@nibirutech.com"
```

## 查看变化
```bash
git diff a.py  # 工作区和暂存区对比
git diff --staged a.py  # 暂存区和版本库对比
```
