---
title: 【Unity】组件批量复制
date: 2021-04-23 18:20:10
tags:
    - Unity
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

Unity Component批量复制

在制作毕设的过程中，我发现我的角色模型需要替换，但是原角色物体上有很多组件，如果全部在Inspector面板右键Copy、Paste很麻烦，所以制作了一个能够将物体上所有组件一键复制粘贴的工具。

![工具窗口](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210423151412982.png)

将原物体放到左边，新物体放到右边，点击检测按钮。

![完成复制](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210423151550651.png)

然后点击Copy按钮，就会把左边物体身上的组件（除了Transform组件和Tag、Layer）复制到右边物体上。

![完成效果](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210423151914716.png)

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEditor;
using UnityEditorInternal;
using UnityEngine;

public class ComponentCopyWindow : EditorWindow
{
    private GameObject rootGO;
    private List<Component> rootComponents;

    private GameObject targetGO;
    private List<Component> targetComponents;

    [MenuItem("Wonderland6627/ComponentCopyWindow")]
    public static void ShowWindow()
    {
        var window = EditorWindow.GetWindowWithRect<ComponentCopyWindow>(new Rect(0, 0, 700, 350));
    }

    private void OnEnable()
    {
        rootComponents = new List<Component>();
        targetComponents = new List<Component>();
    }

    private void OnGUI()
    {
        GUI.Label(new Rect(200, 0, 500, 20), "将'原始物体'的组件都复制到'复制物体上'（不包括Transform、Tag、Layer）");
        GUILayout.Space(20);
        EditorGUILayout.BeginHorizontal();
        {
            GUILayout.Label("原始物体：");
            rootGO = EditorGUILayout.ObjectField(rootGO, typeof(GameObject)) as GameObject;
            if (rootGO)
            {
                EditorGUILayout.BeginVertical();
                {
                    GUILayout.BeginArea(new Rect(5, 50, 300, 300), GUI.skin.box);
                    {
                        if (GUILayout.Button("检测物体组件"))
                        {
                            rootComponents = rootGO.GetComponents<Component>().ToList();
                            rootComponents.RemoveAll((comp) =>
                            {
                                return comp.GetType() == typeof(Transform);
                            });
                        }

                        if (rootComponents != null && rootComponents.Count != 0)
                        {
                            for (int i = 0; i < rootComponents.Count; i++)
                            {
                                GUILayout.Label(i + " " + rootComponents[i].GetType().Name);
                            }
                        }
                    }
                    GUILayout.EndArea();
                }
                EditorGUILayout.EndVertical();
            }

            GUILayout.Space(50);
            if (rootGO && targetGO)
            {
                GUI.enabled = (rootComponents.Count != 0);
                if (GUI.Button(new Rect(310, 160, 70, 30), "Copy to =>", EditorStyles.toolbarButton))
                {
                    for (int i = 0; i < rootComponents.Count; i++)
                    {
                        ComponentUtility.CopyComponent(rootComponents[i]);
                        ComponentUtility.PasteComponentAsNew(targetGO);
                    }

                    targetComponents = targetGO.GetComponents<Component>().ToList();
                    targetComponents.RemoveAll((comp) =>
                    {
                        return comp.GetType() == typeof(Transform);
                    });
                }
                GUI.enabled = true;
            }
            GUILayout.Space(50);

            GUILayout.Label("复制物体：");
            targetGO = EditorGUILayout.ObjectField(targetGO, typeof(GameObject)) as GameObject;
            if (targetGO)
            {
                EditorGUILayout.BeginVertical();
                {
                    GUILayout.BeginArea(new Rect(5 + 80 + 300, 50, 300, 300), GUI.skin.box);
                    {
                        if (targetComponents != null && targetComponents.Count != 0)
                        {
                            for (int i = 0; i < targetComponents.Count; i++)
                            {
                                GUILayout.Label(i + " " + targetComponents[i].GetType().Name);
                            }
                        }
                    }
                    GUILayout.EndArea();
                }
                EditorGUILayout.EndVertical();
            }
        }
        EditorGUILayout.EndHorizontal();
    }
}

```
