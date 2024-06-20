---
title: 【Unity】编辑器扩展 Hierarchy菜单扩展
date: 2021-05-06 18:24:08
tags:
    - Unity
    - Hierarchy Extention
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

在Unity的层级面板选中物体，右键会显示默认菜单，当你定义了自己的预制体，为了方便创建，可以将层级面板菜单扩展，一键添加你的Prefab。

系统默认菜单：
![默认](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210506111403229.jpg)
扩展后的菜单：
![扩展](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210506111504446.jpg)

```csharp
using UnityEngine;
using UnityEditor;

public class TitansHierarchyTools
{
    [MenuItem("GameObject/TitansUI/ImageButton")]
    public static void CreateImageButton()
    {
        GameObject imgBtnRes = AssetDatabase.LoadAssetAtPath<GameObject>("UIPrefabs/ImageButton");
        GameObject imgBtnInstance = GameObject.Instantiate(imgBtnRes);
        imgBtnInstance.name = "ImageButton";

        Transform selection = Selection.activeTransform;
        if (selection != null)
        {
            imgBtnInstance.transform.SetParent(selection);
        }

        Selection.activeGameObject = imgBtnInstance;
    }

    [InitializeOnLoadMethod]
    private static void StartInitializeOnLoadMethod()
    {
        EditorApplication.hierarchyWindowItemOnGUI += OnHierarchyGUI;
    }

    private static void OnHierarchyGUI(int instanceID, Rect selectionRect)
    {
        if (Event.current != null && Event.current.button == 2 && Event.current.type <= EventType.MouseUp)
        {
            if (Selection.activeTransform)
            {
                Vector2 mousePosition = Event.current.mousePosition;
                EditorUtility.DisplayPopupMenu(new Rect(mousePosition.x, mousePosition.y, 0, 0), "GameObject/", null);
            }
        }
    }
}
```
在层级面板中选中一个物体，按下鼠标滚轮键，会出现扩展之后的菜单。并会根据你是否选中了物体，将想要生成的Prefab生成在指定位置。