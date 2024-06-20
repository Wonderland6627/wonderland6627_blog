---
title: 【Unity】鼠标的锁定解锁，隐藏显示，跳转场景鼠标隐藏解决办法
date: 2019-04-05 17:47:14
tags:
    - Unity
    - Cursor
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

```
Cursor.visible = true;//鼠标显示

Cursor.visible = false;//鼠标隐藏

Cursor.lockState = CursorLockMode.Locked;//鼠标锁定并隐藏

Cursor.lockState = CursorLockMode.None;//鼠标解锁并显示

Cursor.lockState = CursorLockMode.Confined;//鼠标限制在Game视图
```

有的时候使用了第一人称控制器再跳转回**开始界面场景**会找不到鼠标，在**开始场景界面**的Start方法中写下这两句即可
```
Cursor.visible = true;
Cursor.lockState = 0;
```
