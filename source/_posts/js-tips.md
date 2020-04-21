---
title: js技巧
date: 2020-04-19
tags: javascript
categories: [javascript]
---

少用以下 5 个 会影响编译优化代码

- for in 改用 Object.keys()
- with
- delete (hidden classes)
- arguments (inline cashing) 改用 ...rest , array from
- eval

![解释器与编译器](https://cdn.jsdelivr.net/gh/aslkami/hexo-blog/source/images/compiler.png)
