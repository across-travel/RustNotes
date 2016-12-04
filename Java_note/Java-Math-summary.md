---
title: Java Math
date: 2017-05-10 21:57:02
category: Java_note
tag: [Java]
---

Java中关于数值的一些问题

### Math.round()的进位问题
`Math.round()`函数对于正数和负数有不同的效果  
对于正数是四舍五入，对于负数是“五舍六入”

```
Math.round(9.4):9, Math.round(9.5):10, Math.round(9.6):10
Math.round(-9.4):-9, Math.round(-9.5):-9, Math.round(-9.6):-10
```