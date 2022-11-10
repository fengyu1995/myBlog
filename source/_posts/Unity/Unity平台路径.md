---
title: Unity平台路径
tags: Unity
abbrlink: adc70860
date: 2022-11-10 16:14:43
---

**Unity3D各平台路径（包括手机内置存储路径、SD卡等等）**



  关于Unity3D在各平台上的路径问题，网上有好多的资料，如下是比较好的参考资料：

1、<http://www.manew.com/thread-23491-1-1.html>

2、<http://www.xuanyusong.com/archives/2656>

  这里我不详细解释和路径的用法，只把各个路径对应的位置和访问方式总结一下。



###### 1、Resources路径

  Resources文件夹是Unity里自动识别的一种文件夹，可在Unity编辑器的Project窗口里创建，并将资源放置在里面。Resources文件夹下的资源不管是否有用，全部会打包进.apk或者.ipa，并且打包时会将里面的资源压缩处理。加载方法是Resources.Load<T>(文件名)，需要注意：文件名不包括扩展名，打包后不能更改Resources下的资源内容，但是从Resources文件夹中加载出来的资源可以更改。

###### 2、Application.dataPath路径

  这个属性返回的是程序的数据文件所在文件夹的路径，例如在Editor中就是项目的Assets文件夹的路径，通过这个路径可以访问项目中任何文件夹中的资源，但是在移动端它是完全没用。

###### 3、Application.streamingAssetsPath路径

  这个属性用于返回流数据的缓存目录，返回路径为相对路径，适合设置一些外部数据文件的路径。在Unity工程的Assets目录下起一个名为“StreamingAssets”的文件夹即可，然后用Application.streamingAssetsPath访问，这个文件夹中的资源在打包时会原封不动的打包进去，不会压缩，一般放置一些资源数据。在PC/MAC中可实现对文件的“增删改查”等操作，但在移动端是一个只读路径。

###### 4、Application.persistentDataPath路径（推荐使用）

  此属性返回一个持久化数据存储目录的路径，可以在此路径下存储一些持久化的数据文件。这个路径可读、可写，但是只能在程序运行时才能读写操作，不能提前将数据放入这个路径。在IOS上是应用程序的沙盒，可以被iCloud自动备份，可以通过同步推送一类的助手直接取出文件；在Android上的位置是根据Project Setting里设置的Write Access路径，可以设置是程序沙盒还是sdcard，注意：如果在Android设置保存在沙盒中，那么就必须root以后才能用电脑取出文件，因此建议写入sdcard里。一般情况下，建议将获得的文件保存在这个路径下，例如可以从StreamingAsset中读取的二进制文件或者从AssetBundle读取的文件写入PersistentDatapath。

###### 5、Application.temporaryCachePath路径

  此属性返回一个临时数据的缓存目录，跟Application.persistentDataPath类似，但是在IOS上不能被自动备份。

###### 6、/sdcard/..路径

  表示Android手机的SD卡根目录。

###### 7、/storage/emulated/0/..路径（这个路径我查找了好久……）

  表示Android手机的内置存储根目录。



  以上各路径中的资源加载方式都可以用WWW类加载，但要注意各个平台路径需要加的访问名称，例如Android平台的路径前要加"jar:file://"，其他平台使用"file://"。以下是各路径在各平台中的具体位置信息：

  Android平台

Application.dataPath :  /data/app/xxx.xxx.xxx.apk

Application.streamingAssetsPath :  jar:file:///data/app/xxx.xxx.xxx.apk/!/assets

Application.persistentDataPath :  /data/data/xxx.xxx.xxx/files

Application.temporaryCachePath :  /data/data/xxx.xxx.xxx/cache

  IOS平台

Application.dataPath :                    Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data

Application.streamingAssetsPath : Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data/Raw

Application.persistentDataPath :    Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents

Application.temporaryCachePath : Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Library/Caches

  Windows Web Player

Application.dataPath :  file:///D:/MyGame/WebPlayer (即导包后保存的文件夹，html文件所在文件夹)

Application.streamingAssetsPath : 

Application.persistentDataPath : 

Application.temporaryCachePath : 