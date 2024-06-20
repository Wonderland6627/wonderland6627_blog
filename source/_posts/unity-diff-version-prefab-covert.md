---
title: 【Unity】预制体转换工具
date: 2021-08-13 18:38:25
tags:
    - Unity
    - Unity Prefab
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

 在Unity开发过程中，会创建很多prefab或者使用一些素材包，由于更换Unity版本或者版本与素材包导出版本不一致，会导致prefab损坏的情况，针对这些情况，我制作了这个预制体转换工具，用于将高版本的预制体转换为低版本Unity可使用的格式。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/8d12624445bf43cab68d7ea79e251364.png)

输入由于版本不一致导致损坏的预制体所在文件夹路径（Assets开头），并点击开始转换。


EditorGUI窗口绘制：

```csharp
using System;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class PrefabConvertWindow : EditorWindow
{
    public static string PrefabDirPath = "Assets/Prefabs";

    [MenuItem("Wonderland6627/PrefabConvertTools/PrefabConvertWindow", priority = 0)]
    public static void OpenWindow()
    {
        EditorWindow.GetWindow<PrefabConvertWindow>();

        PrefabDirPath = PlayerPrefs.GetString("PrefabDirPathSaveKey");
    }

    private void OnGUI()
    {
        GUILayout.Space(10);
        EditorGUILayout.LabelField("预制体文件夹路径(Assets开头)", EditorStyles.boldLabel);
        PrefabDirPath = EditorGUILayout.TextField(PrefabDirPath);

        GUILayout.Space(10);
        if (GUILayout.Button("开始转换"))
        {
            if (string.IsNullOrEmpty(PrefabDirPath) || !PrefabDirPath.StartsWith("Assets") || !Directory.Exists(PrefabDirPath))
            {
                Debug.LogError("路径有误");
                
                return;
            }

            PrefabConvertTools.FindAllPrefabInPath(PrefabDirPath);
        }

        GUILayout.Space(10);
        if (GUILayout.Button("关闭进度条"))
        {
            EditorUtility.ClearProgressBar();
        }
    }

    private void OnLostFocus()
    {
        PlayerPrefs.SetString("PrefabDirPathSaveKey", PrefabDirPath);
    }

    private void OnDisable()
    {
        PlayerPrefs.SetString("PrefabDirPathSaveKey", PrefabDirPath);
    }
}

```
转换方法：

```csharp
public class PrefabConvertTools
{
    public static void FindAllPrefabInPath(string dirPath)
    {
        PlayerPrefs.SetString("PrefabDirPathSaveKey", dirPath);
        string[] allPath = AssetDatabase.FindAssets("t:Prefab", new string[] { dirPath });
        int length = allPath.Length;

        EditorUtility.DisplayProgressBar("处理预制体", "", 0);
        for (int i = 0; i < allPath.Length; i++)
        {
            string path = AssetDatabase.GUIDToAssetPath(allPath[i]);
            if (path.Contains("_Converted"))
            {
                continue;
            }

            var obj = AssetDatabase.LoadAssetAtPath(path, typeof(GameObject)) as GameObject;
            if (obj != null)
            {
                EditorUtility.DisplayProgressBar("处理预制体", obj.name, (float)i / length);
                ConvertPrefab2LowVersion(obj);
            }
        }

        EditorUtility.ClearProgressBar();
        Debug.Log("处理完成");
    }

    /// <summary>
    /// Prefab高版本转换低版本
    /// 带上_Converted后缀来代表转换好的obj
    /// </summary>
    public static void ConvertPrefab2LowVersion(GameObject oldPrefab)
    {
        string rootDirPath = GetObjRootDirPath(oldPrefab, true);
        string newName = oldPrefab.name + "_Converted.prefab";
        PrefabUtility.CreatePrefab(rootDirPath + newName, oldPrefab);
    }

    /// <summary>
    /// 获取文件父文件夹的Assets目录 isSuffix是否带有'/'
    /// </summary>
    private static string GetObjRootDirPath(UnityEngine.Object obj, bool isSuffix = false)
    {
        string fullPath = AssetDatabase.GetAssetPath(obj);
        string dirPath = string.Empty;
        var dirs = fullPath.Split('/');
        for (int i = 0; i < dirs.Length - 1; i++)
        {
            dirPath += dirs[i] + "/";
        }

        return isSuffix ? dirPath : dirPath.Remove(dirPath.Length - 1);
    }
}
```
转换完成之后，会在指定文件夹内生成“原文件名_Converted.prefab"的预制体，即为转换完成的新预制体。
原理大概是Unity的预制体是一种可序列化文件，其格式在不同的Unity版本有微小的不同，而AssetDatabase.LoadAssetAtPath方法能够将文件正常读取，再通过PrefabUtility.CreatePrefab去创建符合当前版本序列化规则的预制体，就能够正常加载不会报prefab broken的警告了。