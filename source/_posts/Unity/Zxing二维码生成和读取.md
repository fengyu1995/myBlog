---
title: Zxing二维码生成和读取
tags: Unity
abbrlink: 41ee7e1f
date: 2022-11-10 16:19:32
---



代码

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using UnityEngine.UI;
using ZXing;
public class QRCodeTest : MonoBehaviour {
    public RawImage cameraTeture;
    public RawImage QRcode;
    public Text text;
    private WebCamTexture webCamTexture;
    private float time;
    BarcodeReader barcodeReader;
    BarcodeWriter barcodeWriter;
    Color32[] data;
	void Start () {
        //除了256  其他不能正确显示
        string str = "http://www.baidu.com";
        ShowQRcode(str, 256,256);
    }
	
	void Update () {
        //time += Time.deltaTime;
        //if (time>=3f)
        //{
        //    QR();
        //    time = 0;
        //}	
	}
//读取二维码
    private void QR()
    {

     barcodeReader = new BarcodeReader();
        WebCamDevice[] devices = WebCamTexture.devices;
        string deviceName = devices[0].name;
        webCamTexture = new WebCamTexture(deviceName,400,300);
        cameraTeture.texture = webCamTexture;
        webCamTexture.Play();
        data = webCamTexture.GetPixels32();
        Result str= barcodeReader.Decode(data, webCamTexture.width,webCamTexture.height);
        if (str!=null)
        {
            text.text = str.Text;
            Debug.Log(str.Text);          
        }
    }

//生成二维码
    void ShowQRcode(string str,int width,int height)
    {
        Texture2D t = new Texture2D(width,height);
         t.SetPixels32(GeneQRCode(str, width, height));
        t.Apply();
        QRcode.texture = t;
        byte[] bytes = t.EncodeToPNG();
        SaveData(bytes);
    }
    Color32[] GeneQRCode(string formatStr,int width,int heught)
    {
        ZXing.QrCode.QrCodeEncodingOptions options = new ZXing.QrCode.QrCodeEncodingOptions();
        options.CharacterSet = "UTF-8";
        options.Width = width;
        options.Height = heught;
        options.Margin = 1;
        barcodeWriter = new BarcodeWriter { Format=ZXing.BarcodeFormat.QR_CODE,Options= options };
      return  barcodeWriter.Write(formatStr);
    }

    void SaveData(byte[] bytes)
    {
        string path = Application.dataPath + @"/Tex.png";
        File.WriteAllBytes(path,bytes);
    }
}
```

