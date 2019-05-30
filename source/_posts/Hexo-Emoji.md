---
title: 让 Hexo 支持 Emoji 表情
author: zranbo
abbrlink: 917c88c8
tags:
  - Hexo
  - ''
categories: []
date: 2019-05-30 11:04:00
---
Hexo 中默认 markdown 渲染器是 hexo-renderer-marked ，这个渲染器是不支持 emoji 表情的。所以我们需要采取一个第三方插件来完成

{% githubCard user:crimx repo:hexo-filter-github-emojis %}

** 安装 **

```bash
npm install hexo-filter-github-emojis --save
```

** 配置 **

打开 `站点配置文件 _config.yml` 并添加以下内容：
```yaml
githubEmojis:
  enable: true
  className: github-emoji
  unicode: true
  styles:
  localEmojis:
```

** 针对 NexT 主题优化 **

修改 `themes/next/layout/_partials/head.swig`

```
algolia: {
  applicationID: '{{ theme.algolia.applicationID }}',
  apiKey: '{{ theme.algolia.apiKey }}',
  indexName: '{{ theme.algolia.indexName }}',
  hits: {{ theme.algolia_search.hits | json_encode }},
  labels: {{ theme.algolia_search.labels | json_encode }}
},
{# 只修改这里添加 emojis 参数 #}
emojis: {
  className: '{{ config.githubEmojis.className | default('github-emoji') }}'
}
```

修改 `themes/next/source/js/src/utils.js`

```js
$('.content img')
  .not('[hidden]')
  .not('.group-picture img, .post-gallery img, img.' + CONFIG.emojis.className) # 修改这里，添加 CONFIG.emojis.className
```

** 使用方法 **

在 [Emoji Cheat Sheet](https://www.webfx.com/tools/emoji-cheat-sheet/) 中找到你想用的表情，点击复制

直接粘贴上面复制的 emoji 编码，例如 `:smile:` 就可以显示对应的 emoji 表情 :smile: