---
title: 避免IDE格式化
date: 2018-06-08 15:59:40
tags:
    -- IDE
    -- formatter
---

## 避免IDE格式化

使用@formatter:off和@formatter:on来包围不需要格式化的代码，IDE就会跳过这段的格式化。

// @formatter:off
...
// @formatter:on