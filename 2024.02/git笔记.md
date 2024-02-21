# GIT笔记

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


## 代码备份
使用git第一个目的：代码备份，备份各个版本随时切换和读取


### 小功能
```bash
git init  # 初始化
git log  # 看版本
git log --pretty=oneline # 简洁
git log --oneline  # 同上
git config --global user.name "Liu Hetian"  # 名字
git config --global user.email "liuhetian@nibirutech.com"  # 邮箱
```

### 查看变化与版本
```bash

git diff HEAD -- <filename>  # 工作区和版本库对比，最重要

git diff <filename>  # 工作区和暂存区对比
git diff --staged <filename>  # 暂存区和版本库对比 也可以 --cached

git log  # 版本
git log --pretty=oneline
```

### 工作区放入暂存区
```bash
git status  -- 查看有哪些文件可以放
git diff <filename>
git add <filename>

# 新版本
git restore --staged <filename>  # 把什么环境变量放进去了，赶紧拿出来；下一次再提交
# 老版本
git reset HEAD <filename>
```

### 从暂存区复原工作区
可能用在睡午觉敲了一堆乱码进去？
```
git status  -- 查看有哪些文件有变化
git diff <filename>
# 老版本
git checkout -- <filename>  # 从暂存区库拉到工作区
# 新版本
git restore <filename>
```

### 暂存区保存到版本库
一个版本应该是可以实现一些功能
```bash
git status
git diff --staged a.py  # 暂存区和版本库对比 也可以 --cached
git commit -m "版本的新功能"

git tag <版本号>
```

### 给版本一个标签
```bash
git tag  # 查看现在有的所有标签
git tag <版本号>  # 给这个版本一个标签，例如版本号

```


### 从版本库复原工作区
如果业务调整，目前所有的代码都不要了

```bash
git diff HEAD -- <filename>  # 工作区和版本库对比，最重要
git diff --staged <filename>  # 暂存区和版本库对比 也可以 --cached
  # 如果只想复原特定的文件
git reset --hard HEAD^  # 全部报废



# 老版本
git reset HEAD <file>
# 新版本
git restore --staged <filename>  # 从版本库拉到暂存区
```

## 分支
```bash
git switch -c <branchname>

```

