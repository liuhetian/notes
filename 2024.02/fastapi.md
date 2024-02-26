# Fastapi
记录一些遇到的问题

## 环境变量与更新
需要用到一个配置表DataFrame，需要从Google Sheet里面读取，然后每个小时定时更新
有`app.state`和全局变量两种形式，似乎差不多
（还没研究明白）

## 寿命事件
启动和关闭时要做一些事情，例如创建/关闭httpx客户端
[文档-寿命事件](https://fastapi.tiangolo.com/advanced/events/)




