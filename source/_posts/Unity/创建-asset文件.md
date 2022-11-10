---
title: 创建.asset文件
tags: Unity
abbrlink: e218aec5
date: 2022-11-10 16:21:09
---



代码

```C#
using UnityEngine;
using System.Collections;
using System;

// 子弹类型枚举
public enum BulletType
{
    DirectAttack = 0,  // 直接攻击
    Phony,             // 假子弹
    Real,              // 真子弹
    Track,             // 追踪子弹
}

/// <summary>
/// 可序列化
/// </summary>
[Serializable]
public class Bullet : ScriptableObject
{

    // Bullet 类直接继承自 ScriptableObject

    // 子弹类型
    public BulletType bulletType = BulletType.DirectAttack;

    // 子弹速度
    public int speed = 10;

    // 伤害数值
    public int damage = 5;

    // 子弹关联的特效
    public GameObject effectObj;
}
```

