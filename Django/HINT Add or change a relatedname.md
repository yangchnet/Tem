---
title: "HINT:Addorchangearelated_name"
date: 2021-01-22T17:27:21+08:00
draft: false
---
## HINT: Add or change a related_name 

解决方案：

需要在setting中重载AUTH_USER_MODEL  

AUTH_USER_MODEL = 'users.UserProfile'  
users：你的app  

UserProfile：model  
