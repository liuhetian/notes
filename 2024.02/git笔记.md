# GIT笔记

2024到来，要优化代码，先从版本控制开始，现在的控制方法很呆：
```
lht_p2_bargain/
├── 1.x
│   ├── 1.0
│   ├── 1.1
│   └── env
├── 2.x
│   ├── 2.0
│   ├── 2.1
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
git config user.name "liuhetian"  # 名字
git config user.email "liuhetian@nibirutech.com"  # 邮箱
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
```

### 从暂存区复原工作区
可能用在睡午觉敲了一堆乱码进去？
```
git status  -- 查看有哪些文件有变化
git diff <filename>
# 老版本
git checkout -- <filename>  # 从暂存区库拉到工作区
# 新版本git
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

### 从版本库复原暂存区
本质上是从版本库复原暂存区，但其实就是删掉暂存区的东西
把什么环境变量放进去了，赶紧拿出来；下一次再提交

```bash
# 复原方法可以查看
git status
git diff --staged a.py  # 暂存区和版本库对比 也可以 --cached

# 老版本
git reset HEAD <filename>
# 新版本
git restore --staged <filename>  
```


### 给版本一个标签
```bash
git tag  # 查看现在有的所有标签
git tag <版本号>  # 给这个版本一个标签，例如版本号

# 可能修个小bug
git tag -d <版本号>
git push -d origin <版本号>
git tag <版本号>
git push origin <版本号>
```

### 使用tag
线上服务器上需要使用不同的版本，切换方法和切换分支一样

```bash
git tag  # 查看有哪些tag
git checkout <版本号>
```

### 从版本库复原工作区
如果业务调整，目前所有的代码都不要了

```bash
git diff HEAD -- <filename>  # 工作区和版本库对比，最重要 如果没有两个斜杠git会优先看是不是一个分支名
git diff --staged <filename>  # 暂存区和版本库对比 也可以 --cached
  # 如果只想复原特定的文件
git reset --hard HEAD^  # 全部报废



# 老版本
git reset HEAD <file>
# 新版本
git restore --staged <filename>  # 从版本库拉到暂存区
```

## 其他功能

### 忽略某些文件
.gitignore文件
```
__pycache__/
.ipynb_checkpoints
.DS_Store
env/
.env
```

#### 删除库中的文件
有可能新建了一个不该被上传到git的文件，例如虚拟环境env/, 或者密码文件.env, 要弥补问题不可能删掉本地文件再git add，这时候需要手动删除版本库里的东西
下面的命令会直接在暂存区和版本库里一起删除，删不删工作区取决于`--cached`选项
```bash
git rm --cached <filename>

# 如果用下面的命令会把工作区的文件也给删掉，感觉不好用
git rm <filename>
```

## 分支
```bash
git switch -c <branchname>

```

