---
title: 'HINT:Addorchangearelated_name'
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# HINT: Add or change a related\_name

解决方案：

需要在setting中重载AUTH\_USER\_MODEL

AUTH\_USER\_MODEL = 'users.UserProfile'  
users：你的app

UserProfile：model

