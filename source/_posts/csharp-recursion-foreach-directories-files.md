---
title: 【C#】递归遍历文件夹下所有文件夹和文件
date: 2021-07-01 18:27:13
tags:
    - C#
    - CSharp
categories:
    - 技术
    - CSharp
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

```csharp
using System;
using System.IO;
using System.Collections.Generic;
using UnityEngine;

public class PathUtils
{
    private static List<DirectoryInfo> directoryInfosList;
    private static List<FileInfo> fileInfosList;

    /// <summary>
    /// 遍历文件及文件夹根目录
    /// </summary>
    /// <param name="rootPath">根文件夹目录</param>
    /// <param name="OnGetDirectoryInfosListResultHandler">文件夹列表获取完成回调</param>
    /// <param name="OnGetFileInfosListResultHandler">文件列表获取完成回调</param>
    /// <param name="debug">是否默认输出信息</param>
    public static void ForeachRootDir(string rootPath, Action<List<DirectoryInfo>> OnGetDirectoryInfosListResultHandler = null, Action<List<FileInfo>> OnGetFileInfosListResultHandler = null, bool debug = false)
    {
    	if (string.IsNullOrEmpty(rootPath))
        {
            return;
        }
        
        directoryInfosList = new List<DirectoryInfo>();
        fileInfosList = new List<FileInfo>();

        DirectoryInfo directoryInfo = new DirectoryInfo(rootPath);
        if (directoryInfo == null)
        {
            return;
        }
        
        ForeachDir(directoryInfo);

        if (debug)
        {
            OnGetDirectoryInfosListResultHandler += DebugList;
            OnGetFileInfosListResultHandler += DebugList;
        }

        if (OnGetDirectoryInfosListResultHandler != null)
        {
            OnGetDirectoryInfosListResultHandler.Invoke(directoryInfosList);
        }

        if (OnGetFileInfosListResultHandler != null)
        {
            OnGetFileInfosListResultHandler.Invoke(fileInfosList);
        }

        if (debug)
        {
            OnGetFileInfosListResultHandler -= DebugList;
            OnGetDirectoryInfosListResultHandler -= DebugList;
        }
    }

    /// <summary>
    /// 遍历文件夹
    /// </summary>
    private static void ForeachDir(DirectoryInfo info)
    {
        foreach (var fileInfo in info.GetFiles())
        {
            fileInfosList.Add(fileInfo);
        }

        foreach (var dirInfo in info.GetDirectories())
        {
            directoryInfosList.Add(dirInfo);
            ForeachDir(dirInfo);
        }
    }

    /// <summary>
    /// 文件夹列表Log回调
    /// </summary>
    private static void DebugList(List<DirectoryInfo> infosList)
    {
        foreach (var info in infosList)
        {
            Debug.Log(info.FullName);
        }
    }

    /// <summary>
    /// 文件列表Log回调
    /// </summary>
    private static void DebugList(List<FileInfo> infosList)
    {
        foreach (var info in infosList)
        {
            Debug.Log(info.FullName);
        }
    }
}

```

遍历D:\UnityProjects\Wonderland6627\AssetDemo\Assets文件夹下所有文件夹和文件，并在控制台输出：
```csharp
PathUtils.ForeachRootDir(@"D:\UnityProjects\Wonderland6627\AssetDemo\Assets", null, null, true);
```
