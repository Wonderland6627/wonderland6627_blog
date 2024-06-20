---
title: 【Unity】动态更改鼠标样式
date: 2021-04-12 18:03:01
tags: 
    - Unity
    - Cursor
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

[方法参考博客](https://gameinstitute.qq.com/community/detail/118295)：https://gameinstitute.qq.com/community/detail/118295

```csharp
using System.Runtime.InteropServices;

public class CursorTools
{
    [DllImport("User32.DLL")]
    public static extern IntPtr LoadCursorFromFile(string fileName);

    [DllImport("User32.DLL")]
    public static extern bool SetSystemCursor(IntPtr hcur, uint id);
    public const uint OCR_NORMAL = 32512;

    [DllImport("User32.DLL")]
    public static extern bool SystemParametersInfo(uint uiAction, uint uiParam, IntPtr pvParam, uint fWinIni);
    public const uint SPI_SETCURSORS = 87;
    public const uint SPIF_SENDWININICHANGE = 2;

	/// <summary>
    /// 把准备好的.cur指针资源放在StreamingAssets/Cursors下，记得创建好这个文件夹
    /// </summary>
    private static string CursorPath(string fileName)
    {
        return Path.Combine(Application.streamingAssetsPath, "Cursors", fileName) + ".cur";
    }

    /// <summary>
    /// 恢复Windows正常鼠标指针
    /// </summary>
    public static void Reset2NormalCursor()
    {
        SystemParametersInfo(SPI_SETCURSORS, 0, IntPtr.Zero, SPIF_SENDWININICHANGE);
    }

    /// <summary>
    /// 设置拖拽移动样式的指针
    /// </summary>
    public static void SetDragCursor()
    {
        IntPtr hcur_drag = LoadCursorFromFile(CursorPath("aero_move"));
        SetSystemCursor(hcur_drag, OCR_NORMAL);
    }
}
```
以拖拽功能为例，想拖拽的时候调用：

```csharp
	CursorTools.SetDragCursor();
```
用完记得把指针变回来：

```csharp
	CursorTools.Reset2NormalCursor();
```
C:\Windows\Cursors文件夹下面有很多Windows默认鼠标资源。