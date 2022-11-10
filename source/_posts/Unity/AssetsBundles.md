---
title: AssetsBundles
tags: Unity
abbrlink: 69537ebe
date: 2022-11-10 16:27:19
---



代码

```C#
首先得写一个Editor类用来打包带有assetsBundle标签的资源，
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;

public class BuildAssetsBundle  {
  
    [MenuItem("BuildAssetsBundleTools/BuildAssetsBundles")]
    public static void  Build()
    {
        string strAbOutPathDIR = string.Empty;
        strAbOutPathDIR = Application.streamingAssetsPath;
        if (!Directory.Exists(strAbOutPathDIR))
        {
            Directory.CreateDirectory(strAbOutPathDIR);
        }
   //打包API  参数：路径 打包设置  打包平台     BuildPipeline.BuildAssetBundles(strAbOutPathDIR,BuildAssetBundleOptions.None,BuildTarget.StandaloneWindows64);
    }
}


还有一个加载类（示例为加载prefab）
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LoadAssetsBundles : MonoBehaviour
{
    public Transform temp;
    // Use this for initialization
    void Start()
    {
        string path = "file://" + Application.streamingAssetsPath + "/prefab";
        StartCoroutine(LoadPrefabAssets(path, "Cube", temp));
    }

    // Update is called once per frame
    void Update()
    {

    }
    IEnumerator LoadPrefabAssets(string path, string assetsName = "", Transform showPos = null)
    {

        if (string.IsNullOrEmpty(path))
            Debug.LogError(GetType() + "/LoadPrefabAssets()/输入参数  path为空，请检查！");
        using (WWW www = new WWW(path))
        {
            yield return www;
            AssetBundle ab = www.assetBundle;
            if (ab != null)
            {
                if (assetsName == "")
                {
                    if (showPos != null)
                    {
                        GameObject tempClonePrefabs = (GameObject)Instantiate(ab.LoadAsset(assetsName));
                        tempClonePrefabs.transform.position = showPos.transform.position;
                    }
                    else
                    {
                        Instantiate(ab.mainAsset);
                    }
                }
                else
                {
                    if (showPos != null)
                    {
                        GameObject tempClonePrefabs = (GameObject)Instantiate(ab.LoadAsset(assetsName));
                        tempClonePrefabs.transform.position = showPos.transform.position;
                    }
                    else
                    {
                        Instantiate(ab.mainAsset);
                    }
                }
            }

        }
    }
}
```

