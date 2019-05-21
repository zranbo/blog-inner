---
title: Hexo 标签插件
author: zranbo
abbrlink: b32026b8
tags:
  - Hexo
categories: []
date: 2019-05-17 23:06:00
---
# Hexo内置

<b>引用块 (blockquote)</b>

- 代码
```
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```

- 效果

{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

---

<b>Pull Quote</b>

- 代码
```
{% pullquote [class] %}
content
{% endpullquote %}
```

- 效果

{% pullquote Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endpullquote %}


# NexT 内置

<b>Bootstrap Callout</b>

- 代码
```
{% note [class] %} Content {% endnote %}
```
 class为一下之一：

 ```
 default
 primary
 success
 info
 warning
 danger
 ```

- 效果

{% note default %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}

{% note primary %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}

{% note success %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}

{% note info %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}

{% note warning %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}

{% note danger %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. {% endnote %}


