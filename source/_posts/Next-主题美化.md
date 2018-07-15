title: Next 主题美化
date: 2018-06-25 19:51:28
categories: 工具
tags: hexo
---

{% cq %} 换上简洁大气的 Next 主题后，总要折腾一番才符合 `猿` 的气质。{% endcq %} 

<!--more-->

## 站内搜索

原文地址：[Local-search](http://theme-next.iissnan.com/third-party-services.html#local-search)

1. 安装 `hexo-generator-searchdb`，在 blog 根目录下执行命令：
   ```
   $ npm install hexo-generator-searchdb --save
   ```

2. 编辑 blog 目录下的配置文件，新增以下内容到任意位置：

   ```
   search:
     path: search.xml
     field: post
     format: html
     limit: 10000
   ```

3. 编辑 主题配置文件，启用本地搜索功能：

   ```
   # Local search
   local_search:
     enable: true
   ```

4. 重新构建就可以看到效果


## 字数统计

参照 [hexo-symbols-count-time](https://github.com/theme-next/hexo-symbols-count-time)



## Next 内置标签

**居中引用**
```
<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="blockquote-center" 是必须的 -->
<blockquote class="blockquote-center"> Content </blockquote>

<!-- 标签 方式，要求版本在0.4.5或以上 -->
{% centerquote %} Content {% endcenterquote %}

<!-- 标签别名 -->
{% cq %} Content {% endcq %}
```
效果：{% cq %} 横眉冷对千夫指，俯首甘为孺子牛。--鲁迅 {% endcq %}



**彩色引用块**

```
{% note className %} 内容块 {% endnote %}
```
className 可以为：`default`  `primary`  `success`  `info`  `warning`  `danger` 

效果：
{% note success %} success {% endnote %}

> markdown 默认