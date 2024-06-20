---
title: 【Unity】使游戏物体按固定路线移动
date: 2019-02-16 17:42:39
tags:
    - Unity
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

适用于物体按固定路线移动（比如迷宫中的怪物寻路），可以先设置一个组存放这些位置。
```
 public Transform[] wayPoints;//目标点组

 private void Update()
 {
      if (transform.position != wayPoints[index].position)
      {
          MovePoints();//未到达index位置，继续移动
      }
      else
      {
          index = ++index % wayPoints.Length;//到达index位置，改变index值
      }
 }

 void MovePoints()
 {
     Vector2 temp = Vector2.MoveTowards(transform.position, wayPoints[index].position, speed);
     //从当前位置移至index位置
     GetComponent<Rigidbody2D>().MovePosition(temp);
     //考虑到有可能会做碰撞检测，所以使用刚体的移动方式
 }
```
