---
title: Unity调用Windows弹提示框
tags: Unity
abbrlink: b83d1b15
date: 2022-11-10 16:05:04
---

**Unity调用Windows弹提示框**





**1.第一个脚本**

```C#
using System;
using System.Runtime.InteropServices;//调用外部库，需要引用命名空间


/// <summary>
/// 为了调用外部库脚本
/// </summary>
public class ChinarMessage
{
    [DllImport("User32.dll", SetLastError = true, ThrowOnUnmappableChar = true, CharSet = CharSet.Auto)]
    public static extern int MessageBox(IntPtr handle, String message, String title, int type);//具体方法
}
```



**2.第二个脚本**

```c#
using System; //引用命名空间下
using UnityEngine;
using UnityEngine.UI;


/// <summary>
/// Windows消息框
/// </summary>
public class ChinarWindowsMessage : MonoBehaviour
{
    private int      returnNumber; //返回值
    private void Start()
    {
            Button();
    }


    private void Button()
    {
           //示例     
 returnNumber = ChinarMessage.MessageBox(IntPtr.Zero, "Chinar-0:返回值均：1", "确认", 0);
       print(returnNumber);
    }
```

//说明

//参数说明 第一个参数不变为 IntPtr.Zero,第二个参数 为中间现实内容,第三个参数为 标题,第四个参数为 类型

//类型:0  按钮为一个确认按钮  显示为确认  返回值为   1

//1   按钮为两个 确认,取消   返回值为1,2

//2   按钮为三个 中止|重试|忽略  返回值为3,4,5

//3   按钮为三个 是 | 否 | 取消  返回值为6,7,2

//4   按钮为两个  是 | 否   返回值为6,7

//5   按钮为两个  重试 | 取消 返回值为4,2

//6   按钮为三个  取消 | 重试 | 继续   返回值为2,10,11      