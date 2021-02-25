---
author: "李昌"
title: "在通过日期获取数据库条目时，出现djangoItmustbeinYYYY-MM-DDHH:MM[:ss[.uuuuuu]][TZ]format.\"]错误"
date: "2021-02-25"
tags: ["Django"]
categories: ["Django"]
ShowToc: true
TocOpen: true
---

## 在通过日期获取数据库条目时，出现 django It must be in YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ] format."]错误

由于日期是从前端获取的，因此将前端日期引用标签改为：{{ comment.time|date:"Y-m-d H:i:s.u"}}
