---
title: "GithubAPI问题"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Github API问题

# 使用github的RESTful API

访问```https://developer.github.com/v3/```来查看帮助文档

## API访问限制

在云服务器上使用git的API时，发现出现```message":"API rate limit exceeded for 59.110.140.133. (But here's the good news: Authenticated requests get a higher rate limit. Check out the documentation for more details.)","documentation_url":"https://developer.github.com/v3/#rate-limiting```提示信息，显然，这是存在着访问限制。

## 查看访问限制

使用```curl -i https://api.github.com/rate_limit```查看自己的限制信息
```json
HTTP/1.1 200 OK
Content-Type: application/json
X-Ratelimit-Limit: 60
X-Ratelimit-Remaining: 59
X-Ratelimit-Reset: 1585470905
Date: Sun, 29 Mar 2020 07:49:35 GMT
Content-Length: 482
Accept-Ranges: bytes
X-GitHub-Request-Id: BB32:30AB:3EF690:50F336:5E80530E

{
  "resources": {
    "core": {
      "limit": 60,
      "remaining": 59,
      "reset": 1585470905
    },
    "graphql": {
      "limit": 0,
      "remaining": 0,
      "reset": 1585471775
    },
    "integration_manifest": {
      "limit": 5000,
      "remaining": 5000,
      "reset": 1585471775
    },
    "search": {
      "limit": 10,
      "remaining": 10,
      "reset": 1585468235
    }
  },
  "rate": {
    "limit": 60,
    "remaining": 59,
    "reset": 1585470905
  }
}
```
rate.limit 为60次，访问60次后就被限制，很显然不太够用

## 提高访问限制
通过在github注册应用程序，可提高到每小时5000次访问，一般应用程序足够了
在```https://github.com/settings/applications/new```注册应用程序  
![](https://www.wangbase.com/blogimg/asset/201904/bg2019042102.jpg)  
就可以得到client_id和client_secret  
使用```curl -u my_client_id:my_client_secret 'https://api.github.com/user/repos'```进行身份验证，然后即可
