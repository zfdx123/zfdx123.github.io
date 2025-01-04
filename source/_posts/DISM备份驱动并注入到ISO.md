---
layout: post
title: DISM备份驱动并注入ISO镜像
date: 2025-01-04 22:54:00
tags: 
    - Win
categories: 
    - Win
license: cc_by_nc_sa
---

{% note red warning %}
请注意修改路径！
{% endnote %}

备份
```cmd
dism /online /export-driver /destination:"D:\DriversBackup"
```

加载镜像文件(解压[ISO->sources->install.win]->创建[mount])
```cmd
dism /mount-wim /wimfile:D:\install.wim /index:1 /mountdir:D:\mount
```

注入驱动
```cmd
dism /image:D:\mount /add-driver /driver:D:\DriversBackup /recurse /forceunsigned
```

检查驱动
```cmd
dism /image:D:\mount /get-drivers
```

提交并卸载镜像
```cmd
dism /unmount-wim /mountdir:D:\mount /commit
```

使用你喜欢的软件(例如:UltraISO)挂载镜像替换 ISO->sources->install.win 为你刚刚修改过的 ***install.win*** 文件即可
