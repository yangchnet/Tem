---
author: "李昌"
title: "VsCode Snippets功能的使用"
date: "2021-03-08"
tags: ["vscode", "snippets"]
categories: ["Tools"]
ShowToc: true
TocOpen: true
---

# VsCode Snippets的Snippets功能
> snippets是代码片段, 在这里的意思是代码模板. 在使用vscode写代码时，有时需要使用代码模板，一个典型的例子是在写文件头注释时，需要一个固定格式的注释，来表明当前的时间、作者等。

## 1. 使用内置的snippets
vscode中已经为我们内置了许多语言的代码模板，在安装了对应的语言插件后,可直接使用这些snippets.
![20210308124044](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210308124044.png)

## 2. 安装来自marketplace的snippets
按`Ctrl+Shift+X`打开marketplace, 输入`@category:"snippets"`,即可下载来自marketplace的snippets
![20210308124320](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210308124320.png)

## 3. 自定义snippets
如果你对内置的或来自marketplace的snippets均不满意,那么你可以自定义你的snippets.  
在File > Preferences >  User Snippets选项下,选择你要定义snippets的文件类型  
![20210308124522](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210308124522.png)
在选择了文件类型之后,你就可以根据vscode提供的Example自定义snippets了.
```
Example:
	"Print to console": {
		"prefix": "log",
		"body": [
			"console.log('$1');",
			"$2"
		],
		"description": "Log output to console"
	
```
`"Print to console"`是你自定义的snippets的名字,`prefix`为前缀,在输入了你定义的`prefix`后,`body`中的内容就会输出到当前光标的位置.  
在`body`中,你可以使用"`variables`"来描述你的snippets, 其格式为:
1. `${1:label}`: 其中的`1`表示在`body`输出后光标会第一个停放在这个位置,而label是对当前`variables`的描述.
2. `${1|one, two, three|}`: 这个语法格式将提醒你选择`one, two, three`中的一个值.  
 ![20210308125550](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210308125550.png)
3. `$name`或`${name:default}`: 其中的name为预定义的变量名,可使用`default`指定其默认值.预定义的变量名有如下:
   > 有关文件与目录的
   > - `TM_SELECTED_TEXT`当前选定的文本或空字符串
   > - `TM_CURRENT_LIN`当前行的内容
   > - `TM_CURRENT_WORD`光标下或空字符串下的单词内容
   > - `TM_LINE_INDEX`基于零指数的行数
   > - `TM_LINE_NUMBER`基于一个索引的行数
   > - `TM_FILENAME`当前文档的文件名
   > - `TM_FILENAME_BASE`没有扩展的当前文档的文件名
   > - `TM_DIRECTORY`当前文档的目录
   > - `TM_FILEPATH`当前文档的完整文件路径
   > - `CLIPBOARD`剪贴板的内容
   > - `WORKSPACE_NAME`打开的工作区或文件夹的名称
   > - `WORKSPACE_FOLDER`打开的工作区或文件夹的路径
  
   > 有关时间的 
   > - `CURRENT_YEAR`本年度
   > - `CURRENT_YEAR_SHORT`今年最后两位数
   > - `CURRENT_MONTH`以两位数表示的月份（例如"02"）
   > - `CURRENT_MONTH_NAME`本月全名（例如"七月"）
   > - `CURRENT_MONTH_NAME_SHORT`本月的简称（例如"七月"）
   > - `CURRENT_DATE`每月的一天
   > - `CURRENT_DAY_NAME`日名称（例如"星期一"）
   > - `CURRENT_DAY_NAME_SHORT`当天的简称（例如"星期一"）
   > - `CURRENT_HOUR`24 小时时钟格式的当前小时
   > - `CURRENT_MINUTE`当前分钟
   > - `CURRENT_SECOND`当前秒
   > - `CURRENT_SECONDS_UNIX`自Unix epoch以来的秒数

   > 有关注释的
   > - `BLOCK_COMMENT_START` Example output: in PHP or in HTML `/*<!--`
   > - `BLOCK_COMMENT_END` Example output: in PHP or in HTML `*/-->`
   > - `LINE_COMMENT` Example output: in PHP `//`



