---
layout: post
title: Hexo模板相关
date: 2024-05-06 14:09:19
# 标签
tags: 
    - Hexo
    - Redefine
# 分类
categories: 
    # 主类
    - Hexo
    # 子类
    - Redefine
# 缩略图
thumbnail: "https://evan.beee.top/img/208184324-f2640ade-587a-4f46-8ad1-7b4c1b31394f.png"
# 文章页头图
cover: "images/logo.svg"
#banner: "图片链接"
# 文章摘要
excerpt: "这是文章摘要 This is the excerpt of the post"
# 时效
expires: 2026-08-31 23:59:59
# 版权信息
# all_rights_reserved：保留所有权利。
# cc_by_nc_sa：署名-非商业性使用-相同方式共享，详情请见 CC BY-NC-SA 4.0。
# cc_by_nc：署名-非商业性使用，详情请见 CC BY-NC 4.0。
# cc_by_nd：署名-禁止演绎，详情请见 CC BY-ND 4.0。
# cc_by_sa：署名-相同方式共享，详情请见 CC BY-SA 4.0。
# cc_by：署名，详情请见 CC BY 4.0。
# public_domain：公有领域，详情请见 CC0 1.0。
license: all_rights_reserved
# 可选补充信息
# copyright: 本文是瞎写的，你如果要用的话，后果自负。
# 置顶
# sticky: 999
mathjax: true # 数学公式
---

# 文件头内容

```
# 标签
tags: 
    - Hexo
    - Redefine
# 分类
categories: 
    # 主类
    - Hexo
    # 子类
    - Redefine
# 缩略图
thumbnail: "https://evan.beee.top/img/208184324-f2640ade-587a-4f46-8ad1-7b4c1b31394f.png"
# 文章页头图
cover: "images/logo.svg"
#banner: "图片链接"
# 文章摘要
excerpt: "这是文章摘要 This is the excerpt of the post"
# 时效
expires: 2026-08-31 23:59:59
# 版权信息
# all_rights_reserved：保留所有权利。
# cc_by_nc_sa：署名-非商业性使用-相同方式共享，详情请见 CC BY-NC-SA 4.0。
# cc_by_nc：署名-非商业性使用，详情请见 CC BY-NC 4.0。
# cc_by_nd：署名-禁止演绎，详情请见 CC BY-ND 4.0。
# cc_by_sa：署名-相同方式共享，详情请见 CC BY-SA 4.0。
# cc_by：署名，详情请见 CC BY 4.0。
# public_domain：公有领域，详情请见 CC0 1.0。
license: all_rights_reserved
# 可选补充信息 (优先级高于license，会覆盖license)
copyright: 本文是瞎写的，你如果要用的话，后果自负。 
# 置顶
sticky: 999
# 数学公式
mathjax: true
```




# 提示块

## 大号提示块

```
{% notel [颜色] [可选: 自定义图标] [标题] %}
内容
支持换行
{% endnotel %}


{% notel default fa-info 信息 %}
换行测试
换行测试
换行测试
{% endnotel %}
 
{% notel blue 提示 %}
换行测试
换行测试
换行测试
{% endnotel %}
 
{% notel red 自定义标题 %}
换行测试
换行测试
换行测试
{% endnotel %}
```

{% notel default fa-info 信息 %}
换行测试
换行测试
换行测试
{% endnotel %}
 
{% notel blue 提示 %}
换行测试
换行测试
换行测试
{% endnotel %}
 
{% notel red 自定义标题 %}
换行测试
换行测试
换行测试
{% endnotel %}

## 小号提示块

```
{% note [样式/颜色] [可选: 自定义图标] %}
笔记内容
{% endnote %}


{% note  %}
默认 提示块标签
{% endnote %}
 
{% note default  %}
default 提示块标签
{% endnote %}
 
{% note primary  %}
primary 提示块标签
{% endnote %}
 
{% note success  %}
success 提示块标签
{% endnote %}
 
{% note info  %}
info 提示块标签
{% endnote %}
 
{% note warning  %}
warning 提示块标签
{% endnote %}
 
{% note danger  %}
danger 提示块标签
{% endnote %}
 
{% note red fa-bolt%}
自定义提示块标签
{% endnote %}
```

{% note  %}
默认 提示块标签
{% endnote %}
 
{% note default  %}
default 提示块标签
{% endnote %}
 
{% note primary  %}
primary 提示块标签
{% endnote %}
 
{% note success  %}
success 提示块标签
{% endnote %}
 
{% note info  %}
info 提示块标签
{% endnote %}
 
{% note warning  %}
warning 提示块标签
{% endnote %}
 
{% note danger  %}
danger 提示块标签
{% endnote %}
 
{% note red fa-bolt%}
自定义提示块标签
{% endnote %}


# Buttons 按钮模块

## 写法

```
{% btn [可选大小]::[名称]::[url]::[可选图标] %}

不设置任何参数的 {% btn 按钮:: / %} 适合融入段落中。
 
regular 按钮适合独立于段落之外：
 
{% btn regular::示例博客::https://www.ohevan.com::fa-solid fa-play-circle %}
 
{% btn regular::示例博客::https://www.ohevan.com::fa-solid fa-play-circle %}
 
large 按钮更具有强调作用，建议搭配 center 使用：
 
{% btn center large::开始使用::https://redefine-docs.ohevan.com::fa-solid fa-download %}
```

{% btn regular::示例博客::https://zfdx123.github.io::fa-solid fa-play-circle %}

### 变量可选值
[可选大小] :
> center, regular, large, center large, center regular

[可选图标] :
> Fontawesome 图标名称，比如 fa-solid fa-house


# Folding 折叠模块

## 效果

{% folding blue::Folding 测试： 点击查看更多 %}
 
啊啊啊啊啊
 
{% note danger  %}
danger 提示块标签
{% endnote %}
 
{% note tip  %}
tip 提示块标签
{% endnote %}
 
{% endfolding %}

## 语法

```
{% folding [颜色]::[标题] %}
 
需要写的内容
 
{% endfolding %}


{% folding blue::Folding 测试： 点击查看更多 %}
 
啊啊啊啊啊
 
{% note danger  %}
danger 提示块标签
{% endnote %}
 
{% note tip  %}
tip 提示块标签
{% endnote %}
 
{% endfolding %}
```

### 颜色列表

> yellow, blue, green, red, orange, pink, cyan, white, black, gray


# Tabs 分栏模块

## 效果

{% tabs First unique name %}
<!-- tab First Tab-->
**This is Tab 1.**
<!-- endtab -->
 
<!-- tab Second Tab-->
**This is Tab 2.**
 
This is Tab 2.
<!-- endtab -->
 
<!-- tab Third Tab-->
**This is Tab 3.**
 
This is Tab 3.
 
This is Tab 3.
<!-- endtab -->
{% endtabs %}


## 写法

使用 Tabs 模块需要在 markdown 中按照以下格式编写：

```
{% tabs 页面内不重复的ID %}
<!-- tab 栏目1名称 -->
内容
<!-- endtab -->
<!-- tab 栏目2名称 -->
内容
<!-- endtab -->
{% endtabs %}
```

其中，页面内不重复的ID 为你为这个选项卡创建的唯一标识符，可以随便取。

每个栏目内容使用 <!-- tab 栏目名称 --> 和 <!-- endtab --> 来定义。

导图 [文档](https://mermaid.js.org/)

```
`mermaid

  graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;
`
```

```mermaid
  graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;
```

公式 [文档](https://github.com/next-theme/hexo-filter-mathjax/)

```
$$
i\hbar\frac{\partial}{\partial t}\psi=-\frac{\hbar^2}{2m}\nabla^2\psi+V\psi
$$

\begin{eqnarray\*}
\nabla\cdot\vec{E}&=&\frac{\rho}{\epsilon_0}\\\\
\nabla\cdot\vec{B}&=&0\\\\
\nabla\times\vec{E}&=&-\frac{\partial B}{\partial t}\\\\
\nabla\times\vec{B}&=&\mu_0\left(\vec{J}+\epsilon_0\frac{\partial E}{\partial t}\right)\\\\
\end{eqnarray\*}

```

$$
i\hbar\frac{\partial}{\partial t}\psi=-\frac{\hbar^2}{2m}\nabla^2\psi+V\psi
$$

\begin{eqnarray\*}
\nabla\cdot\vec{E}&=&\frac{\rho}{\epsilon_0}\\\\
\nabla\cdot\vec{B}&=&0\\\\
\nabla\times\vec{E}&=&-\frac{\partial B}{\partial t}\\\\
\nabla\times\vec{B}&=&\mu_0\left(\vec{J}+\epsilon_0\frac{\partial E}{\partial t}\right)\\\\
\end{eqnarray\*}
