---
title: 在docker中构建django项目
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 在docker中构建django项目

（需安装docker-compose, [安装教程](https://github.com/yangchnet/Tem/tree/73198dc3a08ab7039d303be561dda4e51ef6c3b5/云计算与容器/安装docker-compose.ipynb)）

## 1. 定义项目组件

对于此项目，您需要创建Dockerfile，Python依赖项文件和docker-compose.yml文件。（您可以使用此文件的扩展名.yml或.yaml扩展名。）

### 1.1. 创建一个空目录

该目录应仅包含构建该映像的资源。

### 1.2 创建Dockerfile

内容如下：

```bash
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

[对于DockerFile的解释](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### 1.3 创建requirements.txt

内容如下：

```python
django
django-ckeditor
pillow
numpy
```

### 1.4 创建docker-compose.yml

该docker-compose.yml文件描述了构成应用程序的服务。在此示例中，这些服务是Web服务器和数据库。撰写文件还描述了这些服务使用哪些Docker映像，它们如何链接在一起，以及它们可能需要安装在容器内的任何卷。最后，该docker-compose.yml文件描述了这些服务公开的端口。有关此文件如何工作的更多信息，请参阅[docker-compose.yml参考](https://docs.docker.com/compose/compose-file/)。 内容如下：

```text
version: '3'

services:
  db:
    image: postgres
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
```

## 2 创建django项目

1. 切换到项目跟目录】
2. 通过运行docker-compose run 命令创建django项目

   ```bash
   sudo docker-compose run web django-admin startproject mysite .
   ```

3. 查看项目内容  

   `ls -l`

4. 更改文件所有权  

   ```bash
   sudo chown -R $USER:$USER .
   ```

## 3 连接数据库

1. 打开mysite/setting.py  

   改为：

   ```python
   DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
   }
   ```

2. 从项目的顶级目录运行 docker-compose up命令

