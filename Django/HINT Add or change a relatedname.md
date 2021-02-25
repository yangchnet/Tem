---
author: "李昌"
title: "HINT:Addorchangearelated_name"
date: "2021-02-25"
tags: ["Django"]
categories: ["Django"]
ShowToc: true
TocOpen: true
---

## HINT: Add or change a related_name 

解决方案：

需要在setting中重载AUTH_USER_MODEL  

AUTH_USER_MODEL = 'users.UserProfile'  
users：你的app  

UserProfile：model  
