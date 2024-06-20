---
title: 【Unity】AssetBundle简单打包
date: 2021-07-01 18:30:08
tags:
    - Unity
    - AssetBundle
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

给定工程中一个文件夹，将其下面所有资源打成ab包
遍历文件夹下资源请参考我的另一个文章：
[C# 递归遍历文件夹下所有文件夹和文件（本站）](https://wonderland6627.github.io/技术/CSharp/csharp-recursion-foreach-directories-files/)
[C# 递归遍历文件夹下所有文件夹和文件（CSDN）](https://blog.csdn.net/weixin_42266035/article/details/118383699)
```csharp
using System.IO;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class AssetBundleCreator
{
    [MenuItem("Wonderland6627/BuildBundle")]
    public static void BuildBundle()
    {
        PathUtils.ForeachRootDir(@"D:\UnityProjects\Wonderland6627\AssetDemo\Assets\BuildAssets", null, OnGetFileInfosList);
        BuildPipeline.BuildAssetBundles(Application.dataPath + "/../Bundles/", AllAssetBundleBuildsList.ToArray(),
            BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows64);
    }

    private static List<string> AllBuildAssetPathsList = new List<string>();
    private static List<AssetBundleBuild> AllAssetBundleBuildsList = new List<AssetBundleBuild>();

    private static void OnGetFileInfosList(List<FileInfo> fileInfosList)
    {
        if (fileInfosList == null || fileInfosList.Count == 0)
        {
            Debug.Log("FileInfosList is null");

            return;
        }

        #region 获取所有资源
        List<string> assetPathsList = new List<string>();//目录下所有asset的路径 以Assets开头
        List<string> allBuildAssetPathsList = new List<string>();//需要打包的所有asset路径 包括其依赖 里面不能重复

        string dataPath = Application.dataPath;

        for (int i = 0; i < fileInfosList.Count; i++)
        {
            string fullName = fileInfosList[i].FullName;
            if (fullName.EndsWith(".meta"))
            {
                continue;
            }

            string path = "";
            path = fullName.Substring(dataPath.Length, fullName.Length - dataPath.Length);
            path = "Assets" + path;
            path = path.Replace(Path.DirectorySeparatorChar.ToString(), "/");
            assetPathsList.Add(path);
        }

        for (int i = 0; i < assetPathsList.Count; i++)
        {
            string assetPath = assetPathsList[i];
            if (!allBuildAssetPathsList.Contains(assetPath))
            {
                allBuildAssetPathsList.Add(assetPath);
            }

            string[] depends = AssetDatabase.GetDependencies(assetPath);
            for (int j = 0; j < depends.Length; j++)
            {
                string depend = depends[j];
                if (!allBuildAssetPathsList.Contains(depend))
                {
                    allBuildAssetPathsList.Add(depend);
                }
            }
        }

        //todo 去掉后缀
        #endregion

        #region 创建AssetBundleBuild[]
        List<AssetBundleBuild> allAssetBundleBuildsList = new List<AssetBundleBuild>();
        for (int i = 0; i < allBuildAssetPathsList.Count; i++)
        {
            string assetPath = allBuildAssetPathsList[i];
            if (assetPath.EndsWith(".mat"))
            {
                continue;
            }

            string name = Path.GetFileName(assetPath);

            AssetBundleBuild assetBundleBuild = new AssetBundleBuild();
            assetBundleBuild.assetBundleName = name;
            assetBundleBuild.assetNames = new string[] { assetPath };
            assetBundleBuild.assetBundleVariant = "wonderlandab";

            allAssetBundleBuildsList.Add(assetBundleBuild);
        }
        #endregion

        AllBuildAssetPathsList = allBuildAssetPathsList;
        AllAssetBundleBuildsList = allAssetBundleBuildsList;
    }

    private static void OnGetDirInfosList(List<DirectoryInfo> fileInfosList)
    {

    }
}
```
