---
title: 【Unity】脚本中文乱码，预览显示INVALID_UTF8_STRING解决办法
date: 2020-05-14 17:59:48
tags:
    - Unity
    - Encode
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

**脚本中文乱码的原因可能是.cs文件的编码格式不是UTF-8。**

1.首先查看项目中有没有对脚本进行保存的方法，如果有的话记得转成UTF-8。

2.如果没有的话，可以手动使用记事本另存为UTF-8编码格式。

3.或者可以写一个方法来统一处理：

```csharp
using System;
using System.IO;
using System.Text;
using UnityEditor;

public static class ToUTF8
{
    /// <summary>
    /// 把.cs转成UTF-8格式
    /// </summary>
    [MenuItem("Wonderland6627/Convert2UTF8")]
    public static void Convert2UTF8()
    {
        var dir = "Assets/Scripts/";
        foreach (var f in new DirectoryInfo(dir).GetFiles("*.cs", SearchOption.AllDirectories))
        {
            var s = File.ReadAllText(f.FullName, Encoding.Default);
            try
            {
                File.WriteAllText(f.FullName, s, Encoding.UTF8);
            }
            catch (Exception)
            {
                continue;
            }
        }
    }
}
```
