---
title: ''
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# docker 第一篇

```python
from IPython.display import Image
```

## 1、docker的安装（Ubuntu\)

### 1.1、 设置存储库

> 若是已安装旧版本的docker，  
> 请卸载：sudo apt-get remove docker docker-engine docker.io containerd runc

#### 1.1.1、更新apt索引

```bash
 sudo apt-get update
```

#### 1.1.2、安装依赖

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

#### 1.1.3、添加docker官方的GPG秘钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

> 在进行此步时，出现了sudo: unable to resolve host iZ2ze4512bfzoapfvch6btZ，这是因为机器不能反向解析  
> 打开主机上的 /etc/hosts  
> 添加： 127.0.0.1 【hostname】\# 【hostname】用主机名替代  
> 可在/etc/hostname中修改主机名，sudo shutdown -r now重启过后完成主机名修改

验证添加成功：

```bash
sudo apt-key fingerprint 0EBFCD88
```

#### 1.1.4、 设置存储库

```text
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) \stable"
```

### 1.2、安装dockerCE

#### 1.2.1 更新apt索引

```bash
sudo apt-get update
```

#### 1.2.2、安装最新版本的dockerCE和containerd

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### 1.2.3、通过运行hello-world验证是否正确安装了dockerCE

```bash
sudo docker run hello-world
```

#### 1.2.4、卸载dockerCE

1、卸载dockerCE软件包

```bash
sudo apt-get purge docker-ce
```

2、主机上的图像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷：

```bash
sudo rm -rf /var/lib/docker
```

### 1.3、在docker中运行应用

#### 1.3.1、创建工作目录

```bash
mkdir dockerwork
```

进入：

```bash
cd dockerwork
```

#### 1.3.2、 创建DockerFile

内容如下：

```bash
Use an official Python runtime as a parent image  
FROM python:2.7-slim  

# Set the working directory to /app  
WORKDIR /app  

# Copy the current directory contents into the container at /app  
COPY . /app  

# Install any needed packages specified in requirements.txt  
RUN pip install --trusted-host pypi.python.org -r requirements.txt  

# Make port 80 available to the world outside this container  
EXPOSE 80  

# Define environment variable  
ENV NAME World  

# Run app.py when the container launches  
CMD ["python", "app.py"]
```

有关DockerFile的解释可见[这里](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

> 其中 WORKDIR表示工作目录， COPY是把当前目录下（.）的内容复制到 /app

其中包含两个未建立的文件：requirements.txt & app.py 其内容如下：

> requirements.txt

```text
Flask
Redis
```

> app.py

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket
# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

#### 1.3.3、构建应用程序

```bash
docker build --tag=friendlyhello .
```

#### 1.3.4、运行应用程序

```bash
docker run -p 4000:80 friendlyhello
```

![&#x7ED3;&#x679C;](https://github.com/yangchnet/Tem/tree/73198dc3a08ab7039d303be561dda4e51ef6c3b5/云计算与容器/image/app-in-browser.png)

> 这是运行一般程序的步骤，要构建django项目，请到[这里](https://github.com/yangchnet/Tem/tree/73198dc3a08ab7039d303be561dda4e51ef6c3b5/云计算与容器/在docker中构建django项目.ipynb)

