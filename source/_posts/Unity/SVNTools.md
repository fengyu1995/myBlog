---
title: SVNTools
tags: Unity
abbrlink: 26c59ab3
date: 2022-11-10 16:08:48
---

代码

```C#
using System.Collections;
using System;
using System.Diagnostics;
using UnityEditor;
    internal static class SVNTools
    {
        [MenuItem("Assets/SVN Update", false, 0)]
        private static void SvnUpdate()
        {
            SVNCommand("update");
        }

        [MenuItem("Assets/SVN Commit", false, 0)]
        private static void SvnCommit()
        {
            SVNCommand("commit");
        }

        private static void SVNCommand(string command)
        {
            string path = "Assets";
            string[] assetGuids = Selection.assetGUIDs;
            if (assetGuids != null && assetGuids.Length > 0)
            {
                path = "\"";

                for (int i = 0; i < assetGuids.Length; i++)
                {
                    string assetName = AssetDatabase.GUIDToAssetPath(assetGuids[i]);
                    path += i > 0 ? "*" + assetName : assetName;
                    if (assetName != "Assets")
                    {
                        path += "*" + assetName + ".meta";
                    }
                }

                path += "\"";
            }

            try
            {
                Process process = new Process();
                process.StartInfo.FileName = "TortoiseProc.exe";
                process.StartInfo.Arguments = string.Format("/command:{0} /path:{1}", command, path);
                process.Start();
            }
            catch (Exception exception)
            {
                UnityEngine.Debug.LogWarning(string.Format("Execute SVN process failure, exception message: {0}", exception.Message));
            }
        }
    }
```

