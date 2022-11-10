---
title: C#字符串的中文、数字、字母检测
tags: C#
abbrlink: 2e7ea77f
date: 2022-11-10 16:31:42
---

C#字符串的中文、数字、字母检测

```c#
using System.Text.RegularExpressions;
 
private string GetResult(string str)
{
 int len=str.Length;
 str=Regex.Replace(str,"[a-zA-Z]","");
 string result=(len-str.Length)+"个字母 ";
 len=str.Length;
 str=Regex.Replace(str,"[0-9]","");
 result+=(len-str.Length)+"个数字 ";
 len=str.Length;
 str=Regex.Replace(str,"[\u4e00-\u9fa5]","");
 result+=(len-str.Length)+"个中文";
 return result;
}

```

检测字符串包含中文、英文和数字的个数。

