---
title: Locust---压力测试快速上手
date: 2019-02-19 02:20:41
categories: nodejs
tags:
- nodejs
- 测试
---

# 资料
开发者文档：https://docs.locust.io/en/stable/

## 简介
Locust是一款易于使用的分布式用户负载测试工具。它用于对网站（或其他系统）进行负载测试，并确定系统可以处理多少并发用户。

### 特点
1. 使用python编写测试脚本
2. 支持模拟数十万用户
3. 基于web的操作界面
4. 可以测试任何网站

# 安装
系统：centos 7

```bash
# 安装pip
$ sudo yum -y install epel-release
$ sudo yum -y install python-pip

# 安装 Locust
$ pip install locustio
```

<!--more-->

# 快速上手
项目代码：https://github.com/ddzyan/node-module-example/tree/master/pressure-test

创建文件 locust_test.py
```py
from locust import HttpLocust, TaskSet, task

class koaHttp(TaskSet):
    @task(1)
    def index(self):
        self.client.get("/")

class WebsiteUser(HttpLocust):
    weight = 1
    task_set = koaHttp
    min_wait = 5000
    max_wait = 9000
    host=http://localhost:3000
```

## 注解
### HttpLocust
HttpLocust 类继承 Locust 的类，并为它添加一个客户端属性，它是 HttpSession的一个实例 ，可用于使HTTP请求。
#### 参数
- task_set：指定定义用户行为的类,包含一组任务
- min_wait：最小的等待时间，毫秒
- max_wait：最大的等待时间，毫秒 两者为声明则默认为1秒
- host：加载的主机的URL前缀
- weight：运行的次数比例

### TaskSet
这里我们定义了一个 Locust 的任务，它们是带有一个参数（Locust类实例）的普通Python callables ，这些任务收集在tasks属性的TaskSet类下。
#### 参数
@task(1)：任务装饰器，参数为运行次数的比例

## 启动测试
```bash
$ locust -f ./locust_test.py

[2019-02-19 02:03:23,151] VM_0_14_centos/INFO/locust.main: Starting web monitor at *:8089
[2019-02-19 02:03:23,151] VM_0_14_centos/INFO/locust.main: Starting Locust 0.9.0
```

打开测试界面：http://127.0.0.1:8089
### 设置界面
![image|1901x821](/images/locust-set.png)
#### 参数说明
- Number of users to simulate: 设置模拟用户数
- Hatch rate(users spawned/second): 每秒产生(启动)的虚拟用户数
- 点击 "Start swarming" 按钮，开始运行性能测试

### 测试界面
![image|1901x821](/images/locust-result.png)
#### 参数说明
- Type: 请求的类型，例如GET/POST
- Name: 请求的路径
- request: 当前请求的数量
- fails: 当前请求失败的数量
- Median: 中间值，单位毫秒，一半的服务器响应时间低于该值，而另一半高于该值
- Average: 平均值，单位毫秒，所有请求的平均响应时间
- Min: 请求的最小服务器响应时间，单位毫秒
- Max: 请求的最大服务器响应时间，单位毫秒
- Content Size: 单个请求的大小，单位字节
- reqs/sec: 是每秒钟请求的个数