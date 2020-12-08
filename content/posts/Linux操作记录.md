---
title: "Linux操作记录"
date: 2020-12-01T09:30:17+08:00
draft: true
tags: []
categories: [] 
---

### 查找文件



```bash
find . -name "*.md"
```



要用正则表达式才能匹配到文件



比如这样

```bash
[~/Documents/note/newblog]$ find . -name "*Linux*"                                                                                                                                                                                                 *[main]
./content/posts/Linux操作记录.md
```

