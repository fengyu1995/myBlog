---
title: LineRenderer的使用
tags: Unity
abbrlink: 9b5b2627
date: 2022-11-10 16:01:31
---

核心代码

```C#
       LineRenderer  currentLine = go.AddComponent<LineRenderer>();

       currentLine.material = lineMaterial;//材质设置

       currentLine.startWidth = paintSize;

        currentLine.endWidth = paintSize;//宽度设置

        currentLine.startColor = paintColor://颜色设置

        currentLine.endColor = paintColor;//颜色设置

       currentLine.numCornerVertices = 5;





       private List<Vector3> positions = new List<Vector3>();

        positions.Add(position);

        currentLine.positionCount = positions.Count;

        currentLine.SetPositions(positions.ToArray());//加入点的集合

        lastMousePostion = position;
```

