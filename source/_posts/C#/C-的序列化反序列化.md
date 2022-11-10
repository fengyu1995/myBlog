---
title: C#的序列化反序列化
tags: C#
sticky: 1
abbrlink: b356ea04
date: 2022-11-10 15:06:15
---

​	C#的序列化我总结了两种，一种是xml序列化，另一种是二进制序列化。

#### 1.xml序列化和反序列化

​	xml序列化需要引入命名空间System.Xml.Serialization，先新建一个工具类

Tools.cs，加入序列化和反序列化的方法。

Tools.cs

```c#
        #region Xml序列化和反序列化(需要序列化的类需要添加[Serializable]标记)
        //保存的路径
            public static string baseXmlPath = System.Environment.CurrentDirectory+"/data.xml";
		//序列化保存数据方法
        public static void XmlSaveData(object obj)
        {
            using (Stream stream=new FileStream(baseXmlPath, FileMode.OpenOrCreate,FileAccess.Write))
            {
                using (StreamWriter sw=new StreamWriter(stream))
                {
                    XmlSerializer xmlSerializer = new XmlSerializer(obj.GetType());
                    xmlSerializer.Serialize(sw,obj);
                    Console.WriteLine(baseXmlPath + "文件序列化成功");
                }
            }
        }
       //反序列化获取数据方法
        public static object XmlGetData()
        {
            object obj = new Student();
            using (Stream stream = new FileStream(baseXmlPath, FileMode.Open, FileAccess.Read))
            {
                using (StreamReader sr = new StreamReader(stream,true))
                {
                    XmlSerializer xmlSerializer = new XmlSerializer(obj.GetType());
                    obj = (Student)xmlSerializer.Deserialize(sr);
                }
            }
            return obj;
        }
        #endregion
```

​	接着创建一个数据类DataInfo.cs,创建要序列化的数据类Student,记得序列化的类前加入[Serializable]标记

```c#
    [Serializable]
    public class Student
    {
        public string Name { get; set; }
        public string Age { get; set; }
        public string Grade { get; set; }
    }
```

​	最后在Program进行测试

```C#
        static void Main(string[] args)
        {
            SaveStudent();
            GetStudent();
            Console.ReadKey();
        }


        public  static void SaveStudent()
        {
            Student student = new Student()
            {
                Name = "zhangsan",
                Age = "18",
                Grade="60"
            };
            Tools.XmlSaveData(student);
        }
        public static void GetStudent()
        {
            Student student = (Student)Tools.XmlGetData();
            Console.WriteLine(string.Format("学生的名字为{0},年龄为{1},成绩为{2}",student.Name,student.Age,student.Grade));
        }
```

​	顺利输出，说明测试成功。

#### 2.二进制序列化和反序列化

​	二进制序列化需要引入命名空间using System.Runtime.Serialization.Formatters.Binary，在工具类中加入序列化和反序列化的方法。

Tools.cs

```C#
        #region 二进制序列化和反序列化(需要序列化的类需要添加[Serializable]标记)
        public static string baseBytePath = System.Environment.CurrentDirectory + "/data.txt";
        public static void ByteSaveData(object obj)
        {
            using (Stream stream = new FileStream(baseBytePath, FileMode.OpenOrCreate, FileAccess.Write))
            {
                BinaryFormatter formatter = new BinaryFormatter();
                formatter.Serialize(stream, obj);
                Console.WriteLine(baseBytePath + "文件序列化成功");
            }
        }

        public static object ByteGetData()
        {
            object obj = null;
            using (Stream stream = new FileStream(baseBytePath, FileMode.Open, FileAccess.Read))
            {
                BinaryFormatter formatter = new BinaryFormatter();
                obj = formatter.Deserialize(stream);
            }
            return obj;
        }
        #endregion
```

​	接着继续在DataInfo.cs中创建一个数据类Person

```C#
    [Serializable]
    public class Person
    {
        public string Name { get; set; }
        public string Age { get; set; }
        public string Sex { get; set; }
    }
```

​      最后在Program中测试

```C#
        static void Main(string[] args)
        {

            SavePerson();
            GetPerson();
            Console.ReadKey();
        }
        public static void SavePerson()
        {
            Person person = new Person()
            {
                Name = "lisi",
                Age="15",
                Sex="男"
            };
            Tools.ByteSaveData(person);
        }
        public static void GetPerson()
        {
            Person person = (Person)Tools.ByteGetData();
            Console.WriteLine(string.Format("人物的名字为{0},年龄为{1},性别为{2}", person.Name, person.Age, person.Sex));
        }
```

  测试，顺利输出