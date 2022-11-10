---
title: C#调用windows API的一些方法
tags: C#
abbrlink: fffd1735
date: 2022-11-10 16:33:23
---

[**C#调用windows API的一些方法**](http://www.cnblogs.com/benwu/p/4132026.html)

**使用C#调用windows API（从其它地方总结来的，以备查询）**

C#调用windows API也可以叫做C#如何直接调用非托管代码，通常有2种方法：

1．  直接调用从 DLL 导出的函数。

2．  调用 COM 对象上的接口方法

我主要讨论从dll中导出函数，基本步骤如下：

1．使用 C# 关键字 **static** 和 **extern** 声明方法。

2．将 **DllImport** 属性附加到该方法。**DllImport** 属性允许您指定包含该方法的 DLL 的名称。

3．如果需要，为方法的参数和返回值指定自定义封送处理信息，这将重写 .NET Framework 的默认封送处理。

1．首先我们查询MSDN找到GetShortPathName的原型定义

 

**DWORD** **GetShortPathName(**

  **LPCTSTR** *lpszLongPath***,**

  **LPTSTR** *lpszShortPath***,**

  **DWORD** *cchBuffer*

**);**

 

2．查找对照表进行数据类型的转换（出处：<http://msdn.microsoft.com/msdnmag/issues/03/07/NET/default.aspx?fig=true>  **）**Data

| **Win32 Types**                                              | **Specification**               | **CLR Type**  |
| ------------------------------------------------------------ | ------------------------------- | ------------- |
| char, INT8, SBYTE, CHARâ€                                    | 8-bit signed integer            | System.SByte  |
| short, short int, INT16, SHORT                               | 16-bit signed integer           | System.Int16  |
| int, long, long int, INT32, LONG32, BOOLâ€ , INT             | 32-bit signed integer           | System.Int32  |
| __int64, INT64, LONGLONG                                     | 64-bit signed integer           | System.Int64  |
| unsigned char, UINT8, UCHARâ€ , BYTE                         | 8-bit unsigned integer          | System.Byte   |
| unsigned short, UINT16, USHORT, WORD, ATOM, WCHARâ€ , __wchar_t | 16-bit unsigned integer         | System.UInt16 |
| unsigned, unsigned int, UINT32, ULONG32, DWORD32, ULONG, DWORD, UINT | 32-bit unsigned integer         | System.UInt32 |
| unsigned __int64, UINT64, DWORDLONG, ULONGLONG               | 64-bit unsigned integer         | System.UInt64 |
| float, FLOAT                                                 | Single-precision floating point | System.Single |
| double, long double, DOUBLE                                  | Double-precision floating point | System.Double |
| â€ In Win32 this type is an integer with a specially assigned meaning; in contrast, the CLR provides a specific type devoted to this meaning. |                                 |               |

 

 

 

 

 

3．调用GetShortPathName这个API，简单的写法如下（编译通过的话），

using System;

using *System.Runtime.InteropServices;*

​    public class MSSQL_ServerHandler

​    {

​        [*DllImport*("kernel32.dll")]

​        public *static extern* int GetShortPathName

​        (

​            string path,

​            StringBuilder shortPath,

​            int shortPathLength

)

​     }

而我们之前的例子：

using System;

using System.Runtime.InteropServices;

​    public class MSSQL_ServerHandler

​    {

​        [DllImport("kernel32.dll", CharSet = CharSet.Auto)]

​        public static extern int GetShortPathName

​        (

​            [MarshalAs(UnmanagedType.LPTStr)] string path,

​            [MarshalAs(UnmanagedType.LPTStr)] StringBuilder shortPath,

​            int shortPathLength

)

​     }

对比可知，其中DllImport ，static，extern基本上是必须有的，其他CharSet，MarshalAs（…）是可选项，在这里即使没有，程序也是可以调用此API了。

说明：

1．MSSQL_ServerHandler. GetShortPathName 方法用 **static** 和 **extern** 修饰符声明并且具有 **DllImport** 属性，该属性使用默认名称GetShortPathName 通知编译器此实现来自kernel32.dll。若要对 C# 方法使用不同的名称（如 getShort），则必须在 **DllImport** 属性中使用 **EntryPoint** 选项，如下所示：

**[DllImport("kernel32.dll", EntryPoint="getShort")]**

**2．**使用MarshalAs(UnmanagedType.LPTStr)保证了在任何平台上都会得到LPTStr，否则默认的方式会把从C#中的字符串作为BStr传递。

 

现在如果是仅含有简单参数和返回值的WIN32 API，就都可以利用这种方法进行对照，简单的改写和调用了。

 

**二．背后的原理 ―― 知其所以然，相关的知识**

 **1．平台调用详原理**

平台调用依赖于元数据在运行时查找导出的函数并封送其参数。下图显示了这一过程。

对非托管 DLL 函数的“平台调用”调用

 

 

 

当“平台调用”调用非托管函数时，它将依次执行以下操作：

查找包含该函数的 DLL。

将该 DLL 加载到内存中。

查找函数在内存中的地址并将其参数推到堆栈上，以封送所需的数据。

**注意**   只在第一次调用函数时，才会查找和加载 DLL 并查找函数在内存中的地址。

将控制权转移给非托管函数。

平台调用会向托管调用方引发由非托管函数生成的异常。

 **2．关于Attribute（属性，注意蓝色字）**

属性可以放置在几乎所有声明中（但特定的属性可能限制它在其上有效的声明类型）。在语法上，属性的指定方法为：将括在方括号中的属性名置于其适用的实体声明之前。例如，具有 **DllImport** 属性的类将声明如下：

[DllImport] public class MyDllimportClass { ... }

有关更多信息，请参见 [DllImportAttribute 类](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfSystemRuntimeInteropServicesDllImportAttributeClassTopic.asp)。

许多属性都带参数，而这些参数可以是定位（未命名）参数也可以是命名参数。任何定位参数都必须按特定顺序指定并且不能省略，而命名参数是可选的且可以按任意顺序指定。首先指定定位参数。例如，这三个属性是等效的：

[DllImport("user32.dll", SetLastError=false, ExactSpelling=false)]

[DllImport("user32.dll", ExactSpelling=false, SetLastError=false)]

[DllImport("user32.dll")]

第一个参数（DLL 名称）是定位参数并且总是第一个出现，其他参数为命名参数。在此例中，两个命名参数都默认为假，因此它们可以省略（有关默认参数值的信息，请参见各个属性的文档）。

在一个声明中可以放置多个属性，可分开放置，也可放在同一组括号中：

bool AMethod([In][Out]ref double x);

bool AMethod([Out][In]ref double x);

bool AMethod([In,Out]ref double x);

某些属性对于给定实体可以指定多次。此类可多次使用的属性的一个示例是 [Conditional](http://msdn.microsoft.com/library/CHS/csref/html/vclrfconditional.asp)：

[Conditional("DEBUG"), Conditional("TEST1")] void TraceMethod() {...}

**注意**   根据约定，所有属性名称都以单词“Attribute”结束，以便将它们与 .NET Framework 中的其他项区分。但是，在代码中使用属性时不需要指定属性后缀。例如，[DllImport] 虽等效于 [DllImportAttribute]，但 **DllImportAttribute** 才是该属性在 .NET Framework 中的实际名称。

**3．MarshalAsAttribute 类**

指示如何在托管代码和非托管代码之间封送数据。可将该属性应用于参数、字段或返回值。

该属性为可选属性，因为每个数据类型都有默认的封送处理行为。

大多数情况下，该属性只是使用 [UnmanagedType](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfsystemruntimeinteropservicesunmanagedtypeclasstopic.asp) 枚举标识非托管数据的格式。

例如，默认情况下，公共语言运行库将字符串参数作为 **BStr** 封送到 COM 方法，但是可以通过制定MarshalAs属性，将字符串作为 [LPStr](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfsystemruntimeinteropservicesunmanagedtypeclasstopic.asp)、[LPWStr](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfsystemruntimeinteropservicesunmanagedtypeclasstopic.asp)、[LPTStr](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfsystemruntimeinteropservicesunmanagedtypeclasstopic.asp) 或 [BStr](http://msdn.microsoft.com/library/CHS/cpref/html/frlrfsystemruntimeinteropservicesunmanagedtypeclasstopic.asp) 封送到非托管代码。某些 **UnmanagedType** 枚举成员需要附加信息。

 

 

下面，就让我们写一个小程序，试一试如何用C#语言和DllImport特性来调用Win32 API。

using System;

using System.Runtime.InteropServices;

class Program

{

​      [DllImport("User32.dll")]

​      public static extern int MessageBox(int h, string m, string c, int type);



​      static int Main()

​      {

​           MessageBox(0, "Hello Win32 API", "水之真谛", 4);

​           Console.ReadLine();

​           return 0;

​      }

}





\1. 要使用DllImport这个特性（特性也是一种类），必须使用这一句

using System.Runtime.InteropServices;



\2. 然后我们就可以制造一个DllImport类的实例，并把这个实例绑定在我们要使用的函数上了。“特性类”这种类非常怪——制造类实例的时候不使用MyClass mc = new MyClass();这种形式，而是使用**[特性类(参数列表)]**这种形式；特性类不能独立存在，一定要用作修饰其它目标上（本例是修饰后面的一个函数），不同的特性可以用来修饰类、函数、变量等等；特性类实例在被编译的时候也不产生可执行代码，而是被放进metadata里以备检索。总之，你记住特性类很怪就是了，想了解更多就查查MSDN，懒得查就先这么记——不懂惯性定律不影响你学骑自行车。[DllImport("User32.dll")]是说我们要使用的Win32 API函数在User32.dll这个文件里。问题又来了：我怎么知道那么多API函数都在哪个dll文件里呢？这个你可以在MSDN里查到，位置是**Root->Win32 and COM Development->Development Guides->Windows API->Windows API->Windows API Reference->Functions by Category**。打开这页，你会看到有很多API的分类，API全在这里了。打开一个分类，比如Dialog Box，**在Functions段**，你会看到很多具体的函数，其中就有上面用到的MessageBox函数，点击进入。你将打开MessageBox的详细解释和具体用法。它的名字、返回值、参数类型尽收眼底、一览无余！而且很练英文哦~~~~在这一页的底部，你可以看到一个小表格，里面有一项“Minimum DLL Version   user32.dll”就是说这个函数在user32.dll里。

\3. 接下来就是我们的函数了。在C#里调用Win32函数有这么几个要点。

**第一**：名字要与Win32 API的完全一样。

**第二**：函数除了要有相应的DllImport类修饰外，还要声明成public static extern类型的。

**第三**：也是最变态的一点，函数的返回值和参数类型要与Win32 API完全一致！

常用Win32数据类型与.NET平台数据类型的对应表：

 

| **Win32 Types**                                              | **Specification**               | **CLR Type**  |
| ------------------------------------------------------------ | ------------------------------- | ------------- |
| char, INT8, SBYTE, CHAR                                      | 8-bit signed integer            | System.SByte  |
| short, short int, INT16, SHORT                               | 16-bit signed integer           | System.Int16  |
| int, long, long int, INT32, LONG32, BOOL, INT                | 32-bit signed integer           | System.Int32  |
| __int64, INT64, LONGLONG                                     | 64-bit signed integer           | System.Int64  |
| unsigned char, UINT8, UCHAR, BYTE                            | 8-bit unsigned integer          | System.Byte   |
| unsigned short, UINT16, USHORT, WORD, ATOM, WCHAR, __wchar_t | 16-bit unsigned integer         | System.UInt16 |
| unsigned, unsigned int, UINT32, ULONG32, DWORD32, ULONG, DWORD, UINT | 32-bit unsigned integer         | System.UInt32 |
| unsigned __int64, UINT64, DWORDLONG, ULONGLONG               | 64-bit unsigned integer         | System.UInt64 |
| float, FLOAT                                                 | Single-precision floating point | System.Single |
| double, long double, DOUBLE                                  | Double-precision floating point | System.Double |
| In Win32 this type is an integer with a specially assigned meaning; in contrast, the CLR provides a specific type devoted to this meaning. |                                 |               |

 

以MessageBox函数为例（看刚才给出的函数表），它的Win32原形如下：



int **MessageBox**(HWND *hWnd*,   LPCTSTR *lpText*,    LPCTSTR *lpCaption*,  UINT *uType* );



**函数名**：MessageBox将保持不变。

**返回值**：int 将保持不变（无论是Win32还是C#，int都是32位整数）

**参数表**：H开头意味着是Handle，一般情况下Handld都是指针类型，Win32平台的指针类型是用32位来存储的，所以在C#里正好对应一个int整型。不过，既然是指针，就没有什么正负之分，32位都应该用来保存数值——这样一来，用uint（无符号32位整型）来对应Win32的H类型更合理。不过提醒大家一点，int是受C#和.NET CLR双重支持的，而uint只受C#支持而不受.NET CLR支持，所以，本例还是老老实实地使用了int型。（肚子饿了……再坚持坚持……）

至于LPCTSTR是Long Pointer to Constant String的缩写，说白了就是——字符串。所以，用C#里的string类型就对了。

**修饰符**：要求有相应的DllImport和public static extern



经过上面一番折腾，Win32的MessageBox函数就包装成C#可以调用的函数了：



​    [DllImport("User32.dll")]

​    public static extern int MessageBox(int h, string m, string c, int type);



**第一个**：弹出的MessageBox的父窗口是谁。本例中没有，所以是0，也就是“空指针”。

**第二个**：MessageBox的内容。本例中是“Hello Win32 API”。

**第三个**：MessageBox的标题。本例中用的是本人Blog的名字——水之真谛——请大家不要忘记。

**第四个**：MessageBox上的按钮是什么，如果是0，那就只有一个OK，MessageBox太短了，你将看不全“水之真谛”四个字，于是偶改成了4，这样就有两个按钮了。这些在MSDN的函数用法里都有。不过，我还是非常推荐您阅读一下本人的另一篇拙作**《一个Win32程序的进化》** 。

​     

三. 真的有必要吗？

​        .NET Framework是对Win32 API的良好封装，大部分Win32 API函数都已经封装在了.NET Framework类库的各个类里了。    

下面是个例子

​        那是在很久很久以前，我给一个公司写程序用来控制用户登录，在登录之前，用户不能把鼠标移出登录窗体，因为要控制鼠标，所以我首先想起了调用Win32 API中与Cursor相关的函数来——于是不管三七二十一、花了九牛二虎之力调用了Win32 API中的ClipCursor()这个函数，效果还不错。

​        结果前两天翻.NET Framework类库的时候，发现System.Windows.Forms.Cursor类的Clip属性就是专门做这个用的！差点没把鼻子气歪了……请大家自己动手创建一个C#的Windows程序，把下面的核心代码贴到主窗体的双击事件里，试一试。做这个例子的目的就是要告诉大家：**1.对类库的了解程序直接决定了你编程的效率和质量——用类库里的组件比我们“从轮子造起”要快得多、安全得多。2.不到万不得已，不要去直接调Win32 API函数——那是不安全的。**



Rectangle r = new Rectangle(this.Left, this.Top, this.Width, this.Height);

System.Windows.Forms.Cursor.Clip = r;



最后，大家一定非常想知道，.NET Framework都为我们封装好了哪些Win32 API，OK，MSDN里有一篇文章，专门列出了这些。文章的名字是《Microsoft Win32 to Microsoft .NET Framework API Map》  

 

 

下面以C#为例简单介绍调用API的基本过程：  

 

　 动态链接库函数声明部分一般由下列两部分组成，一是函数名或索引号，二是动态链接库的文件名。  

　 譬如，你想调用User32.DLL中的MessageBox函数，我们必须指明函数的名字MessageBoxA或MessageBoxW，以及库名字User32.dll,我们知道Win32 API对每一个涉及字符串和字符的函数一般都存在两个版本，单字节字符的ANSI版本和双字节字符的UNICODE版本。 



　 下面是一个调用API函数的例子：  

[DllImport("KERNEL32.DLL", EntryPoint="MoveFileW", SetLastError=true,  

CharSet=CharSet.Unicode, ExactSpelling=true,  

CallingConvention=CallingConvention.StdCall)]  

public static extern bool MoveFile(String src, String dst);  



　 其中入口点EntryPoint标识函数在动态链接库的入口位置，在一个受管辖的工程中，目标函数的原始名字和序号入口点不仅标识一个跨越互操作界限的函数。而且，你还可以把这个入口点映射为一个不同的名字，也就是对函数进行重命名。重命名可以给调用函数带来种种便利，通过重命名，一方面我们不用为函数的大小写伤透脑筋，同时它也可以保证与已有的命名规则保持一致，允许带有不同参数类型的函数共存，更重要的是它简化了对ANSI和Unicode版本的调用。CharSet用于标识函数调用所采用的是Unicode或是ANSI版本，ExactSpelling＝false将告诉编译器,让编译器决定使用Unicode或者是Ansi版本。其它的参数请参考MSDN在线帮助. 



在C#中，你可以在EntryPoint域通过名字和序号声明一个动态链接库函数，如果在方法定义中使用的函数名与DLL入口点相同，你不需要在EntryPoint域显示声明函数。否则，你必须使用下列属性格式指示一个名字和序号。 



[DllImport("dllname", EntryPoint="Functionname")]  

[DllImport("dllname", EntryPoint="#123")]  

值得注意的是，你必须在数字序号前加“＃”  

下面是一个用MsgBox替换MessageBox名字的例子：  



using System.Runtime.InteropServices;  



[DllImport("user32.dll", EntryPoint="MessageBox")]  

public static extern int MsgBox(int hWnd, String text, String caption, uint type);  



许多受管辖的动态链接库函数期望你能够传递一个复杂的参数类型给函数，譬如一个用户定义的结构类型成员或者受管辖代码定义的一个类成员，这时你必须提供额外的信息格式化这个类型，以保持参数原有的布局和对齐。 



C#提供了一个StructLayoutAttribute类，通过它你可以定义自己的格式化类型，在受管辖代码中，格式化类型是一个用StructLayoutAttribute说明的结构或类成员，通过它能够保证其内部成员预期的布局信息。布局的选项共有三种： 



布局选项  

描述  

LayoutKind.Automatic  

为了提高效率允许运行态对类型成员重新排序。  

注意：永远不要使用这个选项来调用不受管辖的动态链接库函数。  

LayoutKind.Explicit  

对每个域按照FieldOffset属性对类型成员排序  

LayoutKind.Sequential  

对出现在受管辖类型定义地方的不受管辖内存中的类型成员进行排序。  

传递结构成员  

下面的例子说明如何在受管辖代码中定义一个点和矩形类型，并作为一个参数传递给User32.dll库中的PtInRect函数，  

函数的不受管辖原型声明如下：  

BOOL PtInRect(const RECT *lprc, POINT pt);  

注意你必须通过引用传递Rect结构参数，因为函数需要一个Rect的结构指针。  

using System.Runtime.InteropServices;  



[StructLayout(LayoutKind.Sequential)]  

public struct Point {  

public int x;  

public int y;  

}  



[StructLayout(LayoutKind.Explicit]  

public struct Rect {  

[FieldOffset(0)] public int left;  

[FieldOffset(4)] public int top;  

[FieldOffset(8)] public int right;  

[FieldOffset(12)] public int bottom;  

}  



[DllImport("User32.dll")]  

public static extern Bool PtInRect(ref Rect r, Point p);  

  

类似你可以调用GetSystemInfo函数获得系统信息：  



 using System.Runtime.InteropServices;  

[StructLayout(LayoutKind.Sequential)]  

public struct SYSTEM_INFO {  

public uint dwOemId;  

public uint dwPageSize;  

public uint lpMinimumApplicationAddress;  

public uint lpMaximumApplicationAddress;  

public uint dwActiveProcessorMask;  

public uint dwNumberOfProcessors;  

public uint dwProcessorType;  

public uint dwAllocationGranularity;  

public uint dwProcessorLevel;  

public uint dwProcessorRevision;  

}  



[DllImport("kernel32")]  

static extern void GetSystemInfo(ref SYSTEM_INFO pSI);  



SYSTEM_INFO pSI = new SYSTEM_INFO();  

GetSystemInfo(ref pSI);  



类成员的传递  

同样只要类具有一个固定的类成员布局，你也可以传递一个类成员给一个不受管辖的动态链接库函数，下面的例子主要说明如何传递一个sequential顺序定义的MySystemTime类给User32.dll的GetSystemTime函数, 函数用C/C++调用规范如下: 



void GetSystemTime(SYSTEMTIME* SystemTime);  

不像传值类型,类总是通过引用传递参数.  

 

[StructLayout(LayoutKind.Sequential)]  

public class MySystemTime {  

public ushort wYear;  

public ushort wMonth;  

public ushort wDayOfWeek;  

public ushort wDay;  

public ushort wHour;  

public ushort wMinute;  

public ushort wSecond;  

public ushort wMilliseconds;  

}  



 

[DllImport("User32.dll")]  

public static extern void GetSystemTime(MySystemTime st);  



 

回调函数的传递:  

从受管辖的代码中调用大多数动态链接库函数,你只需创建一个受管辖的函数定义，然后调用它即可,这个过程非常直接。  

如果一个动态链接库函数需要一个函数指针作为参数，你还需要做以下几步：  

首先，你必须参考有关这个函数的文档，确定这个函数是否需要一个回调；第二，你必须在受管辖代码中创建一个回调函数；最后，你可以把指向这个函数的指针作为一个参数创递给DLL函数,. 



**回调函数及其实现:**  

回调函数经常用在任务需要重复执行的场合,譬如用于枚举函数,譬如Win32 API 中的EnumFontFamilies(字体枚举), EnumPrinters(打印机), EnumWindows (窗口枚举)函数. 下面以窗口枚举为例,谈谈如何通过调用EnumWindow 函数遍历系统中存在的所有窗口 



分下面几个步骤:  

\1. 在实现调用前先参考函数的声明  



BOOL EnumWindows(WNDENUMPROC lpEnumFunc, LPARMAM IParam)  



显然这个函数需要一个回调函数地址作为参数.  

\2. 创建一个受管辖的回调函数,这个例子声明为代表类型(delegate),也就是我们所说的回调,它带有两个参数hwnd和lparam,第一个参数是一个窗口句柄，第二个参数由应用程序定义，两个参数均为整形。 



当这个回调函数返回一个非零值时，标示执行成功，零则暗示失败，这个例子总是返回True值，以便持续枚举。



\3. 最后创建以代表对象(delegate)，并把它作为一个参数传递给EnumWindows 函数，平台会自动地 把代表转化成函数能够识别的回调格式。 



using System;  

using System.Runtime.InteropServices;  



public delegate bool CallBack(int hwnd, int lParam);  



public class EnumReportApp {  



[DllImport("user32")]  

public static extern int EnumWindows(CallBack x, int y);  



public static void Main()  

{  

   CallBack myCallBack = new CallBack(EnumReportApp.Report);  

   EnumWindows(myCallBack, 0);  

}  



public static bool Report(int hwnd, int lParam) {  

​    Console.Write("窗口句柄为");  

​    Console.WriteLine(hwnd);  

​    return true;  

}  

}  

**指针类型参数传递**：  

　 在Windows API函数调用时，大部分函数采用指针传递参数，对一个结构变量指针，我们除了使用上面的类和结构方法传递参数之外，我们有时还可以采用数组传递参数。 



　 下面这个函数通过调用GetUserName获得用户名  

BOOL GetUserName(  

​    LPTSTR lpBuffer, // 用户名缓冲区  

​    LPDWORD nSize // 存放缓冲区大小的地址指针  

);  

　   

[DllImport("Advapi32.dll",  EntryPoint="GetComputerName",  ExactSpelling=false,  

SetLastError=true)]  

static extern bool GetComputerName ( [MarshalAs(UnmanagedType.LPArray)] byte[] lpBuffer,  [MarshalAs(UnmanagedType.LPArray)] Int32[] nSize );  



　 这个函数接受两个参数，char * 和int *,因为你必须分配一个字符串缓冲区以接受字符串指针，你可以使用String类代替这个参数类型，当然你还可以声明一个字节数组传递ANSI字符串，同样你也可以声明一个只有一个元素的长整型数组，使用数组名作为第二个参数。上面的函数可以调用如下： 



byte[] str=new byte[20];  

Int32[] len=new Int32[1];  

len[0]=20;  

GetComputerName (str,len);  

MessageBox.Show(System.Text.Encoding.ASCII.GetString(str));  



 

 

[ **C#与C++之间类型的对应**](http://blog.csdn.net/JacksonH/archive/2005/07/27/436410.aspx)

| **Windows Data Type**                                        | **.NET Data Type**           |
| ------------------------------------------------------------ | ---------------------------- |
| BOOL, BOOLEAN                                                | Boolean or Int32             |
| BSTR                                                         | String                       |
| BYTE                                                         | Byte                         |
| CHAR                                                         | Char                         |
| DOUBLE                                                       | Double                       |
| DWORD                                                        | Int32 or UInt32              |
| FLOAT                                                        | Single                       |
| HANDLE (and all other handle types, such as HFONT and HMENU) | IntPtr, UintPtr or HandleRef |
| HRESULT                                                      | Int32 or UInt32              |
| INT                                                          | Int32                        |
| LANGID                                                       | Int16 or UInt16              |
| LCID                                                         | Int32 or UInt32              |
| LONG                                                         | Int32                        |
| LPARAM                                                       | IntPtr, UintPtr or Object    |
| LPCSTR                                                       | String                       |
| LPCTSTR                                                      | String                       |
| LPCWSTR                                                      | String                       |
| LPSTR                                                        | String or StringBuilder*     |
| LPTSTR                                                       | String or StringBuilder      |
| LPWSTR                                                       | String or StringBuilder      |
| LPVOID                                                       | IntPtr, UintPtr or Object    |
| LRESULT                                                      | IntPtr                       |
| SAFEARRAY                                                    | .NET array type              |
| SHORT                                                        | Int16                        |
| TCHAR                                                        | Char                         |
| UCHAR                                                        | SByte                        |
| UINT                                                         | Int32 or UInt32              |
| ULONG                                                        | Int32 or UInt32              |
| VARIANT                                                      | Object                       |
| VARIANT_BOOL                                                 | Boolean                      |
| WCHAR                                                        | Char                         |
| WORD                                                         | Int16 or UInt16              |
| WPARAM                                                       | IntPtr, UintPtr or Object    |
|                                                              |                              |

另： 在进行string转换时，需要加入前缀[MarshalAs(UnmanagedType.LPStr)]

LPDWORD   对应于  ref int

| **C/C++**                                                    | **C#**                                |
| ------------------------------------------------------------ | ------------------------------------- |
| HANDLE, LPDWORD, LPVOID, void*                               | IntPtr                                |
| LPCTSTR, LPCTSTR, LPSTR, char*, const char*, Wchar_t*, LPWSTR | String [in], StringBuilder [in, out]  |
| DWORD, unsigned long, Ulong                                  | UInt32, [MarshalAs(UnmanagedType.U4)] |
| bool                                                         | bool                                  |
| LP<struct>                                                   | [In] ref <struct>                     |
| SIZE_T                                                       | uint                                  |
| LPDWORD                                                      | out uint                              |
| LPTSTR                                                       | [Out] StringBuilder                   |
| PULARGE_INTEGER                                              | out ulong                             |
| WORD                                                         | uInt16                                |
| Byte, unsigned char                                          | byte                                  |
| Short                                                        | Int16                                 |
| Long, int                                                    | Int32                                 |
| float                                                        | single                                |
| double                                                       | double                                |
| NULL pointer                                                 | IntPtr.Zero                           |
| Uint                                                         | Uint32                                |

 

| **Wtypes.h 中的非托管类型** | **非托管 C 语言类型** | **托管类名**                               | **说明**                                                     |
| --------------------------- | --------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| HANDLE                      | void*                 | System.IntPtr                              | 在 32 位 Windows 操作系统上为 32 位，在 64 位 Windows 操作系统上为 64 位。 |
| BYTE                        | unsigned char         | System.Byte                                | 8 位                                                         |
| SHORT                       | short                 | System.Int16                               | 16 位                                                        |
| WORD                        | unsigned short        | System.UInt16                              | 16 位                                                        |
| INT                         | int                   | System.Int32                               | 32 位                                                        |
| UINT                        | unsigned int          | System.UInt32                              | 32 位                                                        |
| LONG                        | long                  | System.Int32                               | 32 位                                                        |
| BOOL                        | long                  | System.Int32                               | 32 位                                                        |
| DWORD                       | unsigned long         | System.UInt32                              | 32 位                                                        |
| ULONG                       | unsigned long         | System.UInt32                              | 32 位                                                        |
| CHAR                        | char                  | System.Char                                | 用 ANSI 修饰。                                               |
| LPSTR                       | char*                 | System.String 或 System.Text.StringBuilder | 用 ANSI 修饰。                                               |
| LPCSTR                      | Const char*           | System.String 或 System.Text.StringBuilder | 用 ANSI 修饰。                                               |
| LPWSTR                      | wchar_t*              | System.String 或 System.Text.StringBuilder | 用 Unicode 修饰。                                            |
| LPCWSTR                     | Const wchar_t*        | System.String 或 System.Text.StringBuilder | 用 Unicode 修饰。                                            |
| FLOAT                       | Float                 | System.Single                              | 32 位                                                        |
| DOUBLE                      | Double                | System.Double                              | 64 位                                                        |

 

BOOL=System.Int32

BOOLEAN=System.Int32

BYTE=System.UInt16

CHAR=System.Int16

COLORREF=System.UInt32

DWORD=System.UInt32

DWORD32=System.UInt32

DWORD64=System.UInt64

FLOAT=System.Float

HACCEL=System.IntPtr

HANDLE=System.IntPtr

HBITMAP=System.IntPtr

HBRUSH=System.IntPtr

HCONV=System.IntPtr

HCONVLIST=System.IntPtr

HCURSOR=System.IntPtr

HDC=System.IntPtr

HDDEDATA=System.IntPtr

HDESK=System.IntPtr

HDROP=System.IntPtr

HDWP=System.IntPtr

HENHMETAFILE=System.IntPtr

HFILE=System.IntPtr

HFONT=System.IntPtr

HGDIOBJ=System.IntPtr

HGLOBAL=System.IntPtr

HHOOK=System.IntPtr

HICON=System.IntPtr

HIMAGELIST=System.IntPtr

HIMC=System.IntPtr

HINSTANCE=System.IntPtr

HKEY=System.IntPtr

HLOCAL=System.IntPtr

HMENU=System.IntPtr

HMETAFILE=System.IntPtr

HMODULE=System.IntPtr

HMONITOR=System.IntPtr

HPALETTE=System.IntPtr

HPEN=System.IntPtr

HRGN=System.IntPtr

HRSRC=System.IntPtr

HSZ=System.IntPtr

HWINSTA=System.IntPtr

HWND=System.IntPtr

INT=System.Int32

INT32=System.Int32

INT64=System.Int64

LONG=System.Int32

LONG32=System.Int32

LONG64=System.Int64

LONGLONG=System.Int64

LPARAM=System.IntPtr

LPBOOL=System.Int16[]

LPBYTE=System.UInt16[]

LPCOLORREF=System.UInt32[]

LPCSTR=System.String

LPCTSTR=System.String

LPCVOID=System.UInt32

LPCWSTR=System.String

LPDWORD=System.UInt32[]

LPHANDLE=System.UInt32

LPINT=System.Int32[]

LPLONG=System.Int32[]

LPSTR=System.String

LPTSTR=System.String

LPVOID=System.UInt32

LPWORD=System.Int32[]

LPWSTR=System.String

LRESULT=System.IntPtr

PBOOL=System.Int16[]

PBOOLEAN=System.Int16[]

PBYTE=System.UInt16[]

PCHAR=System.Char[]

PCSTR=System.String

PCTSTR=System.String

PCWCH=System.UInt32

PCWSTR=System.UInt32

PDWORD=System.Int32[]

PFLOAT=System.Float[]

PHANDLE=System.UInt32

PHKEY=System.UInt32

PINT=System.Int32[]

PLCID=System.UInt32

PLONG=System.Int32[]

PLUID=System.UInt32

PSHORT=System.Int16[]

PSTR=System.String

PTBYTE=System.Char[]

PTCHAR=System.Char[]

PTSTR=System.String

PUCHAR=System.Char[]

PUINT=System.UInt32[]

PULONG=System.UInt32[]

PUSHORT=System.UInt16[]

PVOID=System.UInt32

PWCHAR=System.Char[]

PWORD=System.Int16[]

PWSTR=System.String

REGSAM=System.UInt32

SC_HANDLE=System.IntPtr

SC_LOCK=System.IntPtr

SHORT=System.Int16

SIZE_T=System.UInt32

SSIZE_=System.UInt32

TBYTE=System.Char

TCHAR=System.Char

UCHAR=System.