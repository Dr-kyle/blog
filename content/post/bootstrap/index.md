---
title: "Index"
date: 2022-03-05T16:37:39+08:00
lastmod: 2022-03-05T16:37:39+08:00
draft: false
keywords: ["bootstrap"]
description: "bootstrap study"
tags: ["bootstrap"]
categories: ["bootstrap"]
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

记录 bootstrap 的一些知识

<!--more-->

# Grid

### Grid options

垂直对齐： align-items-center， align-items-start，align-items-end

单独设置某个列的对齐方式：align-self-start，align-self-center， align-self-end

```html
<div class="row">
    <div class="col align-self-start">
```

水平对齐： justify-content-start, justify-content-center,justify-content-end,justify-content-around,justify-content-between(对齐到两边，中间为空)

```html
<div class="row justify-content-start">
    <div class="col-4">
```

|                             | Extra small <576px                   | Small ≥576px | Medium ≥768px | Large ≥992px | Extra large ≥1200px |
| --------------------------- | ------------------------------------ | ------------ | ------------- | ------------ | ------------------- |
| Max container width         | None (auto)                          | 540px        | 720px         | 960px        | 1140px              |
| Class prefix                | `.col-`                              | `.col-sm-`   | `.col-md-`    | `.col-lg-`   | `.col-xl-`          |
| # of columns                | 12                                   |              |               |              |                     |
| Gutter width（.no-gutters） | 30px (15px on each side of a column) |              |               |              |                     |
| Nestable（可嵌套）          | Yes                                  |              |               |              |                     |
| Column ordering             | Yes                                  |              |               |              |                     |

col-md-auto 根据内容自动调整宽度

每行两列，row-cols 快速设置

```html
<div class="row row-cols-1 row-cols-sm-2 row-cols-md-4">
```

强制换行

```html
<div class="row">
    <div class="col-6 col-sm-4">.col-6 .col-sm-4</div>
    <div class="col-6 col-sm-4">.col-6 .col-sm-4</div>

    <!-- Force next columns to break to new line at md breakpoint and up -->
    <div class="w-100 d-none d-md-block"></div>
```

使用`.order-`类来控制内容的**视觉顺序**

```html
<div class="container">
  <div class="row">
    <div class="col order-last">
      First in DOM, ordered last
    </div>
    <div class="col order-4">
      Second in DOM, unordered
   1150 </div>
    <div class="col order-first">
      Third in DOM, ordered first
    </div>
  </div>
</div>
```

偏移列

```html
<div class="row">
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4 offset-md-4">.col-md-4 .offset-md-4</div>
  </div>
```

## 尺寸 Size

w-25 :  25%宽度

h-25：25%的高度

mw-100： max-width: 100%

mh-100：max-height: 100%

相对于视口 viewport 的尺寸

min-vw-100  min-width 100vw

min-vh-100 min-height 100vh

vm-100  width 100vw

vh-100 height 100vh 

## 间隔 Spacing

mx-auto 水平居中

margin padding

- `m` - for classes that set `margin`
- `p` - for classes that set `padding`

Where *sides* is one of:

- `t` - for classes that set `margin-top` or `padding-top`
- `b` - for classes that set `margin-bottom` or `padding-bottom`
- `l` - for classes that set `margin-left` or `padding-left`
- `r` - for classes that set `margin-right` or `padding-right`
- `x` - for classes that set both `*-left` and `*-right`
- `y` - for classes that set both `*-top` and `*-bottom`
- blank - for classes that set a `margin` or `padding` on all 4 sides of the element

Where *size* is one of:

- `0` - for classes that eliminate the `margin` or `padding` by setting it to `0`
- `1` - (by default) for classes that set the `margin` or `padding` to `$spacer * .25`
- `2` - (by default) for classes that set the `margin` or `padding` to `$spacer * .5`
- `3` - (by default) for classes that set the `margin` or `padding` to `$spacer`
- `4` - (by default) for classes that set the `margin` or `padding` to `$spacer * 1.5`
- `5` - (by default) for classes that set the `margin` or `padding` to `$spacer * 3`
- `auto` - for classes that set the `margin` to auto

## Text

text-wrap 让文字换行

text-justify 对齐到组件

text-left, text-center, text-right

text-nowrap 防止文字换行

text-truncate 将文本截断并添加省略号，必须是 display: inline-block 或 display: bolck 类型

text-lowercase, uppercase, capitalize 首字母大写

text-reset 重置文本或链接的颜色，以便从父元素继承颜色属性

text-decoration-none 去除文字的装饰， 不带下换线的链接

## Visibility

仍然占据页面空间

.visible

.invisible