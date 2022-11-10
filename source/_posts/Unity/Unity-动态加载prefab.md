---
title: Unity 动态加载prefab
tags: Unity
abbrlink: 84cd7455
date: 2022-11-10 16:11:04
---

代码

```c#
Object prefab = AssetDatabase.LoadAssetAtPath("Assets/Prefabs/Cube.prefab", typeof(GameObject));  GameObject cube = (GameObject)Instantiate(prefab);
```

