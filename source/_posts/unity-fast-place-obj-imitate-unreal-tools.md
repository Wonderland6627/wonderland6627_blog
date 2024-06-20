---
title: 【Unity】模仿Unreal快速放置物体快捷键工具
date: 2021-04-18 18:10:00
tags:
    - Unity
    - Unreal
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

布置场景的时候，经常会遇到要把物体A放置在物体B表面的情况。
在Unity里面调整起来很麻烦，各种切换视角，调整参数。
而Unreal里面这种情况只需要按下End键，就可以将A完美的放置在B上，所以想在Unity里面也弄一个同样的功能。

![想把这个宝剑放到地上](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210418013429492.png)
![快捷放置之后](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/20210418014144692.png)

快捷放置的实现方法如下：

```csharp
	[MenuItem("Wonderland6627/PlaceTools/PlaceBox")]
    public static void PlaceBoxOnFloor()
    {
        Transform transform = Selection.activeTransform;
        if (transform == null)
        {
            Debug.LogError("NoSelectionTransform");
            
            return;
        }

        bool isCollider = true;//是否自带碰撞器，有的话不需要移除
        BoxCollider collider = transform.GetComponent<BoxCollider>();
        if (collider == null)
        {
            isCollider = false;
            collider = transform.gameObject.AddComponent<BoxCollider>();
        }

        Ray ray = new Ray(transform.position, Vector3.down);
        RaycastHit hit;
        if (Physics.Raycast(ray, out hit))
        {
            if (hit.collider)//向下的射线所遇到的物体有碰撞器
            {
                Debug.Log("On the " + hit.collider.name);
                if (transform.localScale.y == 1)//物体y轴上没有缩放，碰撞器的高度数据准确
                {
                    float offsetY = collider.size.y / 2;
                    float targetPosY = hit.point.y + offsetY;
                    Vector3 offsetVec = new Vector3(transform.localPosition.x, targetPosY, transform.localPosition.z);
                    transform.localPosition = offsetVec;
                }
                else
                {
                    Debug.LogError("Y!=1");
                }
            }
            else
            {
                Debug.LogError("No Plane");
            }
        }

        if (!isCollider)
        {
            Object.DestroyImmediate(collider);
        }
    }
```

原理很笨，利用了射线检测，对A下方发射射线，计算自己应该落到的坐标。
大多情况下，Unity都能给物体计算出完美的BoxCollider，但这种方法也只能应付简单的环境。
以后毕业有空了再对复杂的环境多做一些处理。O(∩_∩)O

```csharp
	[MenuItem("Wonderland6627/PlaceTools/PlaceObject")]
    public static void PlaceObjectOnFloor()
    {
        Transform transform = Selection.activeTransform;
        if (transform == null)
        {
            Debug.LogError("NoSelectionTransform");
            
            return;
        }

        MeshFilter[] meshFilters = transform.GetComponentsInChildren<MeshFilter>();
        SkinnedMeshRenderer[] skinnedMeshRenderers = transform.GetComponentsInChildren<SkinnedMeshRenderer>();
        List<Vector3> modelVerticesList = new List<Vector3>();

        if (meshFilters.Length + skinnedMeshRenderers.Length == 0)
        {
            Debug.LogError("No MeshFilter or SkinnedMeshRenderer");
            
            return;
        }

        //找到y最小的部件
        List<Transform> partsList = new List<Transform>();
        for (int i = 0; i < meshFilters.Length; i++)
        {
            partsList.Add(meshFilters[i].transform);
        }
        for (int i = 0; i < skinnedMeshRenderers.Length; i++)
        {
            partsList.Add(skinnedMeshRenderers[i].transform);
        }

        //只找位置最低的这个
        Transform lowestGO = null;
        PlaceToolsHelper.FindLowest(partsList, ref lowestGO);
        if (lowestGO == null)
        {
            Debug.LogError("No lowest");
            
            return;
        }

        Object part = lowestGO;
        Mesh partMesh = null;

        if (lowestGO.GetComponent<MeshFilter>())
        {
            part = lowestGO.GetComponent<MeshFilter>();
            partMesh = (part as MeshFilter).sharedMesh;
        }
        else if (lowestGO.GetComponent<SkinnedMeshRenderer>())
        {
            part = lowestGO.GetComponent<SkinnedMeshRenderer>();
            partMesh = (part as SkinnedMeshRenderer).sharedMesh;
        }
        if (partMesh == null)
        {
            Debug.LogError("No partMesh");
            
            return;
        }

        var modelVertices = partMesh.vertices;
        for (int i = 0; i < modelVertices.Length; i++)
        {
            if (!modelVerticesList.Exists(vec => vec == modelVertices[i]))
            {
                modelVerticesList.Add(modelVertices[i] + lowestGO.position);
            }
        }

        /*//全找
        List<Mesh> modelMeshesList = new List<Mesh>();//mesh的vertices可能是相对自身center的
        for (int i = 0; i < meshFilters.Length; i++)
        {
            modelMeshesList.Add(meshFilters[i].sharedMesh);
        }

        for (int i = 0; i < skinnedMeshRenderers.Length; i++)
        {
            modelMeshesList.Add(skinnedMeshRenderers[i].sharedMesh);
        }

        for (int i = 0; i < modelMeshesList.Count; i++)
        {
            var vs = modelMeshesList[i].vertices;
            for (int j = 0; j < vs.Length; j++)
            {
                if (!modelVerticesList.Exists(vec => vec == vs[j]))
                {
                    modelVerticesList.Add(vs[j]);
                } 
            }
        }*/

        Vector3 lowestVec = default(Vector3);
        PlaceToolsHelper.FindLowest(modelVerticesList, ref lowestVec);

        Ray ray = new Ray(lowestVec, Vector3.down);
        RaycastHit hit;
        if (Physics.Raycast(ray, out hit))
        {
            if (hit.collider)//向下的射线所遇到的物体有碰撞器
            {
                Debug.Log("On the " + hit.collider.name);
                Vector3 centerOffsetVec = lowestGO.root.position - lowestVec;
                float offsetY = Mathf.Abs(lowestVec.y - hit.point.y);
                float targetPosY = lowestGO.position.y - offsetY;
                Vector3 offsetVec = new Vector3(transform.localPosition.x, targetPosY, transform.localPosition.z) + centerOffsetVec;
                transform.localPosition = offsetVec;
            }
            else
            {
                Debug.LogError("No Plane");
            }
        }
    }
```

PlaceToolsHelper用来辅助找到网格里面最下面的点

```csharp
public class PlaceToolsHelper
{
    public static void FindLowest(List<Transform> transformList, ref Transform lowest)
    {
        if (transformList.Count == 0)
        {
            Debug.LogError("Empty List");
            return;
        }

        lowest = transformList[0];
        for (int i = 0; i < transformList.Count; i++)
        {
            if (transformList[i].position.y < lowest.position.y)
            {
                lowest = transformList[i];
            }
        }
    }

    public static void FindLowest(List<Vector3> vectorList, ref Vector3 lowest)
    {
        if (vectorList.Count == 0)
        {
            Debug.LogError("Empty List");
            return;
        }

        lowest = vectorList[0];
        for (int i = 0; i < vectorList.Count; i++)
        {
            if (vectorList[i].y < lowest.y)
            {
                lowest = vectorList[i];
            }
        }
    }
}
```
最后在EditorWindow的OnGUI方法中监听End按键，就可以实现和Unreal放置物体一样的效果了。
```csharp
		Event e = Event.current;
        if (e.keyCode == KeyCode.End)
        {
            if (e.type == EventType.KeyDown)
            {
                if (!isEndDown)
                {
                    isEndDown = true;
                    PlaceTools.PlaceObjectOnFloor();
                }
            }
            else if (e.type == EventType.KeyUp)
            {
                isEndDown = false;
            }
        }
```
