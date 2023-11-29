---
title: "**CSV** 线路格式"
linktitle: "CSV 线路格式"
weight: 1
---

➟ [速查目录...]({{< ref "/routes/csv_quick/_index.md" >}}) 

## ■ 内容目录

{{% contents %}}

- [1. Overview](#overview)
- [2. Syntax](#syntax)
- [3. Preprocessing](#preprocessing)
- [4. The Options namespace](#options)
- [5. The Route namespace](#route)
- [6. The Train namespace](#train)
- [7. The Structure namespace](#structure)
- [8. The Texture namespace](#texture)
- [9. The Cycle namespace](#cycle)
- [10. The Signal namespace](#signal)
- [11. The Track namespace](#track)
  - [11.1. Rails](#track_rails)
  - [11.2. Geometry](#track_geometry)
  - [11.3. Objects](#track_objects)
  - [11.4. Stations](#track_stations)
  - [11.5. Signalling and speed limits](#track_signalling)
  - [11.6. Safety systems](#track_safety)
  - [11.7. Miscellaneous](#track_misc)

{{% /contents %}}

## <a name="overview"></a>■ 1. 综述

CSV格式线路是以纯文本形式编辑的线路。

线路文件是纯文本，且可以使用任意[字符编码]({{< ref "/information/encodings/_index.md" >}})，但是，我们推荐线路作者们使用UTF8-BOM编码格式。游戏在解析数字数据时，使用的[解析方法]({{< ref "/information/numberformats/_index.md" >}})是 **宽松** 的(特别指出处除外)，但是，我们推荐线路作者们*严格规范*地编写线路文件。 这个文件一般来说可以被放在*Train*或*Railway*文件夹下的任何地方。文件名可以随意，但扩展名必须是 **.csv** 。线路文件被从头到尾逐行解析，每行被分割并被从左到右解析。

路线文件包括一系列指令，用来导入线路中用到的模型（Structure（结构）命名空间），线路的属性信息（说明线路使用的默认列车和背景等信息），文件的其余部分是Track（轨道）命名空间。在这一部分中，主轨道位置（一般以米为单位）被用来描述轨道在哪里转弯，车站的位置在哪里，墙壁应当在哪里开始、哪里结束，以此类推。说得明白一点儿，Track（轨道）命名空间的指令是要放在最后的。

这个格式预设了一条游戏默认的主轨道（0号轨道），不可以指定它开始的位置，也不能结束它。和游戏中其他轨道不同的是，它从线路的开始一直延续到线路的终点，代表着玩家驾驶的列车行驶的轨道。除此之外，游戏中定义的其他轨道都是只供装饰，不能行驶的。不过可以使用 [轨道循迹物件]({{< ref "/routes/xml/trackfollowingobject/_index.md" >}}) 来让AI列车在其它轨道上行驶。

可以几何意义上地弯曲和抬升默认的主轨道，而其他轨道都是相对于主轨道定义的，并随主轨道弯曲起伏。除非特别修改定义，线路中每25米划分为一个区间块，特定的命令只有在区间块的边界位置（整25米位置）才能发挥作用。物体的放置总是基于一个坐标系，它的轴并不弯曲，而是直直地指向邻近的下一个区间块。  

➟ [还可查阅这份CSV格式的速查目录...]({{< ref "/routes/csv_quick/_index.md" >}})

## <a name="syntax"></a>■ 2. 语法

对于线路文件中的每一行，在开头和结尾的[空格]({{< ref "/information/whitespaces/_index.md" >}})会被统统忽略。然后，每一行指令都会按逗号(U+002C，英文半角的那个)分割。于是，每一行都会被看作这样一个格式：

{{% command %}}    
*表达式<sub>1</sub>*, *表达式<sub>2</sub>*, *表达式<sub>3</sub>*, ..., *表达式<sub>n</sub>*    
{{% /command %}}

表达式内容主要有以下类别：

##### ● 注释

注释就是给人看的，游戏完全不管的。以分号(U+003B，英文半角)开头的表达式都会被视为注释。

##### ● 主轨道位置与距离

{{% command %}}  
*位置数字*  
{{% /command %}}    
一个非负的 [严格格式]({{< ref "/information/numberformats/_index.md" >}}) 浮点数，代表一个主轨道位置。随后的指令都将以此位置作为基准点。

{{% command %}}    
*部分<sub>1</sub>*:*部分<sub>2</sub>*:...:*部分<sub>n</sub>*    
{{% /command %}}      
这是一个配合Options.UnitOfLength的更加复杂的距离指定格式，可用于非公制计量单位。每一个 *部分<sub>i</sub>* 都是 [严格格式]({{< ref "/information/numberformats/_index.md" >}}) 的浮点数。 *部分<sub>1</sub>* 会被乘上 *系数<sub>1</sub>*, *部分<sub>2</sub>* 乘上 *系数<sub>2</sub>*，以此类推，真正的主轨道位置是所有积的和。这个结果必须是非负的。各部分被冒号(U+003A，英文半角)分隔。如果想了解如何设定系数，请参见Options.UnitOfLength一节。

在任何参数中使用距离的命令中，这个冒号表示法就可以被使用，到时我们会用<font color="green">绿色</font>标出这种情况。

当*n*个单位系数被使用Options.UnitOfLength定义，但是使用冒号表示法时输入的部分却没有全部给出，那么这些系数将会被向右匹配，在左边的会被忽略。因此，这几种表示方法是等效的：*0:0:2*，*0:2*，和*2*.

##### ● 指令

没有参数的指令：

{{% command %}}    
*指令名称*    
{{% /command %}}

含有几个参数的指令：

{{% command %}}    
*指令名称* *参数<sub>1</sub>*;*参数<sub>2</sub>*;*参数<sub>3</sub>*;...;*参数<sub>n</sub>*    
*指令名称*(*参数<sub>1</sub>*;*参数<sub>2</sub>*;*参数<sub>3</sub>*;...;*参数<sub>n</sub>*)  
{{% /command %}}

不仅有参数还有后缀和编号的指令：

{{% command %}}    
*指令名称*(*编号<sub>1</sub>*;*编号<sub>2</sub>*;...;*编号<sub>m</sub>*) *参数<sub>1</sub>*;*参数<sub>2</sub>*;*参数<sub>3</sub>*;...;*参数<sub>n</sub>*    
*指令名称*(*编号<sub>1</sub>*;*编号<sub>2</sub>*;...;*编号<sub>m</sub>*).*后缀* *参数<sub>1</sub>*;*参数<sub>2</sub>*;*参数<sub>3</sub>*;...;*参数<sub>n</sub>*    
*指令名称*(*编号<sub>1</sub>*;*编号<sub>2</sub>*;...;*编号<sub>m</sub>*).*后缀*(*参数<sub>1</sub>*;*参数<sub>2</sub>*;*参数<sub>3</sub>*;...;*参数<sub>n</sub>*)  
{{% /command %}}

规则：

*指令名称*是大小写都可以的。编号和参数被分号(U+003B，英文半角)隔开。在*指令名称*、编号和参数周围的空格都是被忽略的，括号周围的空格也是被忽略的。

如果要使用编号，它必须被成对括号(U+0028，英文半角在键盘9上面)和(U+0029，英文半角在键盘0上面)括起来。在使用编号时至少要提供一个参数和一个*后缀*。

有两个使用参数的方法变种，除了$开头的预处理指令($Chr, $Rnd, $Sub, ...)之外，可以按照个人喜好二选一。  
第一种方法：参数被至少一个空格(U+0020，平常用的那个)和编号、指令的大名以及*后缀*分开。  
第二种方法：参数被成对括号(U+0028，英文半角在键盘9上面的那个)和(U+0029，英文半角在键盘0上面的那个)括起来。  
在第二种方法中，当使用编号时就必须使用*后缀*。在参数周围的空格都会被忽略。

请注意在有些指令中，不管是用哪种表示方法，*后缀*都是必需的。在接下来的文档中，必需的*后缀*将被 **加粗** ，对于第一种方法加不加均可的后缀将被使用<font color="gray">灰色</font>表示。

##### ● **With** 语句

{{% command %}}    
With *命名空间前缀*    
{{% /command %}}

不严谨地来说，在CSV路线中，“命名空间”的意思和“章节”、“部分”差不多。  
其后所有直接以点号 (U+002E，英文半角) 开头的指令都被自动加上 *命名空间前缀* 。 例如：

{{% code %}}    
With Route    
.Gauge 1435    
.Timetable 1157_M    
{{% /code %}}

和这个是等效的：

{{% code %}}    
Route.Gauge 1435    
Route.Timetable 1157_M    
{{% /code %}}

## <a name="preprocessing"></a>■ 3. 预处理

在游戏开始解析线路文件之前，预处理指令将会被预先处理替换。预处理器会按照正常顺序分析这些以$开头的预处理命令。$Chr、$Rnd和$Sub指令可以随便嵌套，$Include、$If、$Else和$EndIf不能出现在另一个指令的括号里。

{{% warning-nontitle %}}

预处理指令的语法不可以随意使用，必须以下面给出的形式出现。

{{% /warning-nontitle %}}

---

{{% command %}}    
$Include(*文件*)    
$Include(*文件*:*主轨道位置偏移量*)    
$Include(*文件<sub>1</sub>*; *权值<sub>1</sub>*; *文件<sub>2</sub>*; *权值<sub>2</sub>*; ...)    
{{% /command %}}

{{% command-arguments %}}    
***文件<sub>i</sub>***: 要被导入本线路的另一个同格式线路文件 (CSV/RW)。   
***权值<sub>i</sub>***: 一个正浮点数，表示对应的这个文件被使用的可能性大小。数越大，这个文件就越可能被随机选中。    
{{% /command-arguments %}}

该指令按照权值随机选出一个线路文件，再将其内容导入本线路中。因为该文件内容会被不加修改直接在该指令的位置插入，可能需要照顾一下With指令等，不要让它们出现冲突。如果参数中最后一个文件没有给出权值，它会被按1处理。

$Include指令在几个线路有大量重复内容时很有用，可以把重复内容单独存进另一个文件，然后在主文件中导入它，以方便编码。这个指令还可以被用来随机选取线路代码。请注意导入的文件中也可以包含$Include指令来引用更多的文件，但是应该避免循环引用，例如A导入B而B又导入A。要导入的那个文件不应该使用.csv作为格式扩展名（也许用.include是个方便区分的好主意），不然玩家可能会一不小心从主菜单选择了这个“不完全的线路文件”然后发现没法加载（除非那个文件本身就是一个单独的线路，然后本来就想让玩家加载它）。

如果任何一个*文件<sub>i</sub>*被一个:*主轨道位置偏移量*后缀，那个文件中所有的主轨道位置表达式都会被按照这个量**以米作单位**偏移。例如，$Include(stations.include:2000) 会将stations.include文件中的所有内容在插入前都向前偏移2000米。需要注意这些主轨道位置表达式是在所有的预处理命令都被执行完才会被做偏移处理。这意味着像"1$Rnd(2;8)00"这样的主轨道位置表达式尽管在预处理前还不是一个主轨道距离表达式，但是它的随机结果照样会被进行偏移处理。

{{% warning-nontitle %}}

只有OpenBVE1.2.11版以上支持“主轨道位置偏移量”。

{{% /warning-nontitle %}}

---

{{% command %}}    
$Chr(*Ascii编号*)    
{{% /command %}}

{{% command-arguments %}}  
***Ascii编号***: 一个在10~13或20~127范围内的数，代表一个拥有对应ASCII码的字符。  
{{% /command-arguments %}}

这个指令会在原位插入一个对应*Ascii码*的字符。如果想要在某个地方放置一个字符却又不想破坏指令语法结构（比如站名里开头带空格、带括号逗号分号等，如果不这样加入就会被游戏误读），可以使用这个指令。有关的字符有：

{{% table %}}

| Ascii码 | 含义             | 对应字符 |
| ---- | ------------------- | --------- |
| 10   | 换行 (CR)             | *换行* |
| 13   | 换行 (LF)             | *换行* |
| 20   | 空格               | *空格*   |
| 40   | 括号 | (         |
| 41   | 回括号 | )         |
| 44   | 逗号               | ,         |
| 59   | 分号           | ;         |

{{% /table %}}

"$Chr(13)$Chr(10)"(CRLF)代表一次换行。插入$Chr(59)可能基于它的位置被解释为注释开始或指令参数分隔符。

---

{{% command %}}    
$Rnd(*最小值*; *最大值*)    
{{% /command %}}

{{% command-arguments %}}  
***最小值***：一个整数，代表随机数可以取的最小值。  
***最大值***：一个整数，代表随机数可以取的最大值。  
{{% /command-arguments %}}

这个指令会在原位插入一个位于*最小值*和*最大值*之间的随机整数。可以用这个来给线路添加一些随机性。

{{% code "*Example for the use of the $Rnd directive:*" %}}    
10$Rnd(3;5)0, Track.FreeObj 0; 1    
{{% /code %}}

会产生以下三个结果之一：  
{{% code "*Possible outcome from the previous example is exactly __one__ of these lines:*" %}}    
1030, Track.FreeObj 0; 1    
1040, Track.FreeObj 0; 1    
1050, Track.FreeObj 0; 1    
{{% /code %}}

---

{{% command %}}    
$Sub(*编号*) = *表达式*    
{{% /command %}}

{{% command-arguments %}}    
***编号***: 一个非负整数，代表要存储的变量编号。   
***表达式***: 要存进这个变量的数字的值。  
{{% /command-arguments %}}

这个指令只应该单独出现，它将会把*表达式*的值赋给编号为*编号*的这个变量。它不在原位插入任何内容。可以把一个随机数存起来然后以后多次使用（比如说放置几棵到一条随机但是却相同的轨道的排成一排的树）。下面关于 $Sub(*编号*) 的介绍处有几个使用实例。

{{% warning %}}

#### 程序实现备注

虽然变量中也可以存储非数字的内容，还是不能把逗号通过$Chr(44)存进去然后希望它在使用时会起分隔符的作用。但是，可以把分号通过$Chr(59)存进去然后把它调用时放在开头使那一行成为注释。不过因为可以使用$Include和$If来进行条件判断，所以并没有必要这么干。

{{% /warning %}}

---

{{% command %}}  
$Sub(*编号*)  
{{% /command %}}

{{% command-arguments %}}  
***编号***：一个非负整数，代表要读出的变量编号。  
{{% /command-arguments %}}

这个指令会在原位插入编号为*编号*的变量的内容。在读取前这个变量必须先被赋值。

{{% code "*Example for the use of the $Sub(Index)=Value and $Sub(Index) directives:*" %}}    
$Sub(0) = $Rnd(3; 5)    
1000, Track.FreeObj $Sub(0); 47    
1020, Track.FreeObj $Sub(0); 47    
1040, Track.FreeObj $Sub(0); 47    
{{% /code %}}

在这个例子中，三个Track.FreeObj指令都使用同样的随机数值，所以这三个物体会被放在同一条随机的轨道边。如果使用三个$Rnd(3;5)而不是$Sub，这三个物体可能被单独放在不同的轨道边。

---

{{% command %}}  
$If(*条件*)  
{{% /command %}}

{{% command-arguments %}}  
***条件***：一个数值。如果它等于0，则它代表**不成立**的情况。如果它不等于0，则代表**成立**的情况。  
{{% /command-arguments %}}

$If(如果……那么)指令让游戏只在这个条件成立，即为非零值时才解析下面的线路指令。$If的后面必须有一个$EndIf。在$If和$EndIf之间也可以加一个$Else来表示条件是不成立的时要解析的指令。

---

{{% command %}}    
$Else()    
{{% /command %}}

$Else(否则)指令只在前面的$If的条件是不成立的时才解析下面的线路指令。

---

{{% command %}}    
$EndIf()    
{{% /command %}}

$EndIf指令标识前面的$If指令的影响范围到此结束。

{{% code "*Example of $If, $Else and $EndIf*" %}}    
$Sub(1) = 0    
With Track    
$If($Sub(1))    
&nbsp;&nbsp;&nbsp;&nbsp;1020, .FreeObj 0; 1    
&nbsp;&nbsp;&nbsp;&nbsp;1040, .FreeObj 0; 2    
$Else()    
&nbsp;&nbsp;&nbsp;&nbsp;1030, .FreeObj 0; 3    
$EndIf()    
{{% /code %}}

{{% code "*Another example of $If, $Else and $EndIf*" %}}    
With Track    
1050    
$If($Rnd(0;1)), .FreeObj 0; 4, $Else(), .FreeObj 0; 5, $EndIf()    
{{% /code %}}

可以嵌套$If指令。如果把$If/$Else/$EndIf放在不同的几行上，可以更清晰地表示块结构并使它更易读（例如第一个例子）。

---

**最后**，当预处理结束，线路中的所有指令都会被按照轨道位置重新排序。

{{% code "*Example of a partial route file:*" %}}    
1000, Track.FreeObj 0; 23    
1050, Track.RailType 0; 1    
10$Rnd(3;7)0,  Track.Rail 1; 4    
Track.FreeObj 1; 42    
{{% /code %}}

经过预处理之后(假设随机结果是3):  
{{% code "*Preprocessing the $Rnd directive (assuming 3 is produced):*" %}}    
1000, Track.FreeObj 0; 23    
1050, Track.RailType 0; 1    
1030, Track.Rail 1; 4    
Track.FreeObj 1; 42    
{{% /code %}}

最后被按照顺序重新排序:  
{{% code "*Sorting by associated track positions:*" %}}    
1000, Track.FreeObj 0; 23    
1030, Track.Rail 1; 4    
Track.FreeObj 1; 42    
1050, Track.RailType 0; 1    
{{% /code %}}

## <a name="options"></a>■ 4. Options (选项) 命名空间

这个命名空间里的指令设置一些基本设置，并影响其他指令的实际处理效果。所以应当在开始编写写其他指令之前先使用这些指令把设置都调整妥当。

---

{{% command %}}  
**Options.UnitOfLength** *与米的换算系数<sub>1</sub>*; *与米的换算系数<sub>2</sub>*; *与米的换算系数<sub>3</sub>*; ...; *与米的换算系数<sub>n</sub>*  
{{% /command %}}

{{% command-arguments %}}  
***与米的换算系数<sub>i</sub>***：一个浮点数，表示一个单位等于多少米。*与米的换算系数1*的默认值是1，其他的默认值都是0。  
{{% /command-arguments %}}

这个指令可以被用来改变其他指令使用的长度单位。当和形如*部分<sub>1</sub>*:*部分<sub>2</sub>*:...:*部分<sub>n</sub>*的主轨道位置一起使用时，*部分<sub>i</sub>*会被乘上*系数<sub>i</sub>*，以此类推，真正的主轨道位置是所有积的和。更改了长度单位时，您也应同时使用 Options.BlockLength 将区间块长度也设为相应单位。

换算系数的几个示例：

{{% table %}}

| 单位 | 换算系数 |
| ------------ | ----------------- |
| 哩/英里         | 1609.344          |
| 冈特测链        | 20.1168           |
| 米/公尺        | 1                 |
| 码         | 0.9144            |
| 呎/英尺         | 0.3048            |

{{% /table %}}

在下面的示例中，会被Options.UnitOfLength影响的指令参数将被用<font color="green">绿色</font>表示。

---

{{% command %}}  
**Options.UnitOfSpeed** *与千米每时的换算系数*  
{{% /command %}}

{{% command-arguments %}}  
***与千米每时的换算系数***：一个浮点数，表示一个单位等于多少千米每时。默认值是1。  
{{% /command-arguments %}}

这个指令可以被用来改变其他指令使用的速度单位。换算系数的几个示例：

{{% table %}}

| 单位    | 换算系数 |
| --------------- | ----------------- |
| 米每秒   | 3.6               |
| 英里每时      | 1.609344          |
| 千米每时 | 1                 |

{{% /table %}}

在下面的示例中，会被Options.UnitOfSpeed影响的指令参数将被用<font color="blue">蓝色</font>表示。

---

{{% command %}}  
**Options.BlockLength** <font color="green">*长度*</font>  
{{% /command %}}

{{% command-arguments %}}  
***长度***：一个正浮点数，表示区间块的长度，**默认**的单位是**米**。默认值是25米。  
{{% /command-arguments %}}

这个指令可以设置区间块的长度。这是个全局设置，一旦设置就不能来回更改。如果Options.UnitOfLength在该指令前被调用过，*长度*会使用Options.UnitOfLength作为单位。

---

{{% command %}}  
**Options.ObjectVisibility** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：游戏判断一个物体是否可视采用的算法。默认值是0（原版）。  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 原版：当该物体所在的区间块已经被玩家通过，游戏就认为这个物体不再可见，也不再加载显示这个物体。游戏不会显示任何玩家后方的物体。在朝前看时不会有任何问题，但当向后看时所有物体都会消失，且没有任何途径可解决。自相交的轨道（例如环线）不会被正确显示。该设置会造成不正确且无法解决的视觉效果，保留且默认采取该设置只是为了兼容一些老线路。请不要在新线路中采用这个设置。  
**1**: 基于轨道：游戏仍然会加载列车后方可视范围内的物体。这是按照轨道位置计算的。可惜自相交的轨道（例如环线）还是不会被正确显示。推荐添加这个设置。  
{{% /command-arguments %}}

---

{{% command %}}  
**Options.SectionBehavior** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：Track.Section指令应当被如何理解。默认值是0（默认）。  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 默认：在Track.Section *状态<sub>0</sub>*; *状态<sub>1</sub>*; ...; *状态<sub>n</sub>*中，任何一个*状态<sub>i</sub>*都代表当一个闭塞区间的前方有*i*个闭塞区间清空时要传达给信号机的信号状态。  
**1**: 简化：在Track.Section *状态<sub>0</sub>*; *状态<sub>1</sub>*; ...; *状态<sub>n</sub>*中，每个闭塞区间的信号状态都是比它前面一个闭塞区间的状态值要高的最小值。  
{{% /command-arguments %}}

---

{{% command %}}  
**Options.CantBehavior** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：Track.Curve指令应当被如何理解。默认值是0（忽略符号）  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 忽略符号：Track.Curve中的*CantInMillimeters*参数是无符号的，符号会被忽略，轨道总是会向弯道中心倾斜来提供弯道向心力。轨道不会在直道段倾斜。  
**1**: 保留符号：Track.Curve中的*CantInMillimeters*参数是有符号的，正值会使轨道向弯道中心倾斜，负值会使轨道向弯道外侧倾斜。轨道会在直道段倾斜，正值使轨道右倾，负值使轨道左倾。  
{{% /command-arguments %}}

---

{{% command %}}  
**Options.FogBehavior** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：Track.Fog命令应当被如何理解。默认值是0（基于区间块）  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 基于区间块：雾的颜色和范围从原先Track.Fog设置时的区间块的到新Track.Fog设置时的区间块以线性插值平滑过渡。  
**1**: 线性：雾的颜色和范围从两次Track.Fog调用的位置之间线性插值平滑过渡。这和Track.Brightness的效果类似。  
{{% /command-arguments %}}

---

{{% command %}}  
**Options.CompatibleTransparencyMode** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：当一个物体的材质使用的颜色通道不是真彩色时该如何判断透明区域。这可以在游戏设置中调整，但该指令可以超控游戏设置中的选项。  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 特定匹配：如果该图像使用的颜色通道色板内不包括设定的透明色，那么这个图象就会按不透明来处理。  
**1**: 模糊匹配：如果该图像使用的颜色通道色板内不包括设定的透明色，那么就转而匹配色板内与此最接近的颜色。  
{{% /command-arguments %}}

---

{{% command %}}  
**Options.EnableBveTsHacks** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：是否应用几项对于部分奇葩的BVETs2/4线路的兼容性设置。这可以在游戏设置中调整，但该指令可以超控游戏设置中的选项。  
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**: 不启用兼容模式。  
**1**: 启用兼容模式。  
{{% /command-arguments %}}

## <a name="route"></a>■ 5. Route (路线) 命名空间

这个命名空间里的指令设置一些线路的基本信息。

---

{{% command %}}  
**Route.Comment** *文字*  
{{% /command %}}

{{% command-arguments %}}  
***文字***：选择线路页面上显示的描述文字。   
{{% /command-arguments %}}

{{% warning-nontitle %}}  
如果需要插入换行、逗号之类的字符，必须使用$Chr。  
{{% /warning-nontitle %}}

---

{{% command %}}  
**Route.Image** *文件*  
{{% /command %}}

{{% command-arguments %}}  
***文件***：选择线路时显示的描述图片，路径按相对于存储这一线路文件的文件夹书写。   
{{% /command-arguments %}}

---

{{% command %}}  
**Route.Timetable** *文字*  
{{% /command %}}

{{% command-arguments %}}  
***文字***：作为线路默认时刻表表头的文字。   
{{% /command-arguments %}}

{{% warning-nontitle %}}  
如果需要插入换行、逗号之类的字符，必须使用$Chr。  
{{% /warning-nontitle %}}

---

{{% command %}}  
**Route.Change** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：当游戏启动时，列车的车载信号系统默认处于什么状态。   
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**-1**：车载信号系统**已被启动**，制动手柄处于*最大常用制动*位置。  
**0**：车载信号系统**已被启动**，制动手柄处于**紧急制动**位置。  
**1**：车载信号系统*未被设置好*，制动手柄处于**紧急制动**位置。  
{{% /command-arguments %}}

---

{{% command %}}  
**Route.Gauge** *毫米数值*  
{{% /command %}}

{{% command-arguments %}}  
***毫米数值***：一个浮点数，代表轨道的轨距。单位是**毫米**（0.001m,0.1cm）。默认值是1435（标准轨）。   
{{% /command-arguments %}}

{{% note %}}

Route.Gauge和Train.Gauge作用相同。

{{% /note %}}

---

{{% command %}}  
**Route.Signal**(*状态编号*)<font color="gray">.Set</font> <font color="blue">*允许速度*</font>  
{{% /command %}}

{{% command-arguments %}}  
***状态编号***：一个非负整数，代表信号状态。0代表前方没有区间空闲（红灯），1代表1个区间空闲，以此类推。  
<font color="blue">***速度***</font>：一个非负整数，代表这个信号状态下允许的最大通过速度。**默认的**单位是**千米每时**。  
{{% /command-arguments %}}

用这个指令来设定信号状态所允许的最大通过速度。状态0代表前方没有区间空闲（红灯），1代表1个区间空闲，以此类推。默认值（根据默认的日式信号系统设置）是：

{{% table %}}

| *状态编号* | 显示                                                       | 允许速度       |
| ------- | ------------------------------------------------------------ | ----------- |
| 0       | <font color="#C00000">●</font>                               | 0 km/h      |
| 1       | <font color="#FFC000">●●</font>                              | 25 km/h     |
| 2       | <font color="#FFC000">●</font>                               | 55 km/h     |
| 3       | <font color="#00C000">●</font><font color="#FFC000">●</font> | 75 km/h     |
| 4       | <font color="#00C000">●</font>                               | *没有限制* |
| 5       | <font color="#00C000">●●</font>                              | *没有限制* |

{{% /table %}}

---

{{% command %}}  
**Route.RunInterval** *间隔<sub>0</sub>*; *间隔<sub>1</sub>*; ...; *间隔<sub>n-1</sub>*  
{{% /command %}}

{{% command-arguments %}}  
***间隔<sub>i</sub>***：一个浮点数，指定一辆AI列车从第一站的发车时间和玩家操作的列车的时间差。单位是**秒**。正数使AI列车在玩家之前发车，负数使AI列车在玩家之后发车。   
{{% /command-arguments %}}

该指令创建几辆和玩家在同一轨道上的AI车。这些列车可见，正常工作，且使用和玩家一样的列车模型。其他列车使用和玩家相同的时刻表，但是发车时间和玩家列车有着*间隔<sub>i</sub>*的时间差。通过Track.Sta指令，可以为玩家和AI车指定不同的停车站。在玩家列车之后的车辆只会在区间里没有列车时出现，但玩家列车不论是否已经有列车阻挡位置都会被放置。所以，应当正确设置在玩家之前发车的AI车的发车时间，使玩家进入游戏时该列车已经离开玩家要出现的轨道区域。

{{% note %}}

Route.RunInterval和Train.Interval作用相同。

{{% /note %}}

---

{{% command %}}  
**Route.AccelerationDueToGravity** *数值*  
{{% /command %}}

{{% command-arguments %}}  
**Value**: A positive floating-point number representing the acceleration due to gravity in **meters per second squared** (m/s2). The default value is 9.80665.  
{{% /command-arguments %}}

---

{{% command %}}  
**Route.Elevation** <font color="green">*高度数值*</font>  
{{% /command %}}

{{% command-arguments %}}  
<font color="green">***高度数值***</font>：一个浮点数，表示主轨道起始点的海拔高度。**默认**的单位是**米**。默认值是0。   
{{% /command-arguments %}}

---

{{% command %}}  
**Route.Temperature** *摄氏度数值*  
{{% /command %}}

{{% command-arguments %}}
***ValueInCelsius***: 代表 **攝氏** 的初始溫度(浮點數)。 預設值為20。 
{{% /command-arguments %}}

---

{{% command %}}  
**Route.Pressure** *千帕数值*  
{{% /command %}}

{{% command-arguments %}}  
***千帕数值***：一个正浮点数，表示以**千帕斯卡**为单位的环境大气压力。默认值是101.325（标准大气压）。   
{{% /command-arguments %}}

---

{{% command %}}  
**Route.DisplaySpeed** *单位名称,换算系数*  
{{% /command %}}

{{% command-arguments %}}  
***单位名称***：当显示关于速度的信息时，使用的单位名称。  
***换算系数***：一个千米每时相当于几个单位。例如对于英里每时，换算系数是0.621371。  
{{% /command-arguments %}}

该指令可以改变关于速度的信息（例如超速提示）中使用的单位。 

---

{{% command %}}  
**Route.LoadingScreen***文件*  
{{% /command %}}

{{% command-arguments %}}  
***文件***：一个路径，指向一个图像文件。  
{{% /command-arguments %}}

该指令可以自定义加载界面的背景图。 

---

{{% command %}}  
**Route.StartTime** *时间*  
{{% /command %}}

{{% command-arguments %}}  
***时间***：线路载入，模拟开始的时间。  
{{% /command-arguments %}}

该指令可以定义玩家进入游戏时的时间。

{{% note %}}

如果开始时间没有用这一指令特意设定或设定的值不正确，游戏将自动采用第一站的到达时间。 

{{% /note %}}

---

{{% command %}}  
**Route.DynamicLight** *动态光XML配置文件*  
{{% /command %}}

{{% command-arguments %}}  
***动态光XML配置文件***：一个路径，指向配置动态光照的XML文件。  
{{% /command-arguments %}}

该指令可以被用来替代*Route.AmbientLight*，*Route.DirectionalLight*和*Route.LightDirection*。

它允许根据时间改变光照，具体细节在这一页上说明：

[Dynamic Lighting]({{< ref "/routes/xml/dynamiclight/_index.md" >}})

---

{{% command %}}  
**Route.AmbientLight** *红色分量*; *绿色分量*; *蓝色分量*  
{{% /command %}}

{{% command-arguments %}}  
***红色分量***：一个0~255之间的整数，表示漫射环境光颜色的红色值。默认值是160。  
***绿色分量***：一个0~255之间的整数，表示漫射环境光颜色的绿色值。默认值是160。：一个0~255之间的整数，表示漫射环境光颜色的红色值。默认值是160。  
***蓝色分量***：一个0~255之间的整数，表示漫射环境光颜色的蓝色值。默认值是160。  
{{% /command-arguments %}}

该指令指定漫射环境光的颜色。这个光照亮场景中所有物体的所有面。

---

{{% command %}}  
**Route.DirectionalLight** *红色分量*; *绿色分量*; *蓝色分量*  
{{% /command %}}

{{% command-arguments %}}  
***红色分量***：一个0~255之间的整数，表示定向照射光颜色的红色值。默认值是160。  
***绿色分量***：一个0~255之间的整数，表示定向照射光颜色的绿色值。默认值是160。  
***蓝色分量***：一个0~255之间的整数，表示定向照射光颜色的蓝色值。默认值是160。  
{{% /command-arguments %}}

该指令指定定向照射光的颜色。只有物体的迎光面会被照亮，背光面不受影响。应配合*Route.LightDirection*使用，设定光照方向。

---

{{% command %}}  
**Route.LightDirection** *θ 倾斜角*; *φ 方位角*  
{{% /command %}}

{{% command-arguments %}}  
***θ 倾斜角***：一个以**角度**为单位的浮点数，控制定向光的仰角。默认值是60。  
***φ 方位角***：一个以**角度**为单位的浮点数，控制定向光的方位。默认值是-26.57。  
{{% /command-arguments %}}

该指令定义相对于主轨道位置0的定向光照射方向。这是光照亮区域的计算方向，和光源（太阳）的方向正好相反。首先，*θ 倾斜角*定义光照仰角。90代表垂直向下，-90代表竖直向上，在这两个极值处*φ*就不影响位置。 *θ*值为0时代表从地平线后方水平向前照射。*φ 方位角*表示一个旋转方向。*φ*为0代表从前方，90为从正右，-90为正左。分别设置*θ*和*φ*为(180,0)或(0,180)都代表从后方照射。通过调节这两个数值，可以精确地控制光照的方向。

![__光照方向示意图](/images/illustration_light_direction.png)

将球面坐标(θ,φ)换算为直角坐标(x,y,z)的公式：  
{{% function "*Converting a spherical direction (theta,phi) into a cartesian direction (x,y,z):*" %}}    
x = cos(theta) * sin(phi)    
y = -sin(theta)    
z = cos(theta) * cos(phi)    
{{% /function %}}

{{% function "*Converting a cartesian direction (x,y,z) into a spherical direction (theta,phi) for y2≠1:*" %}}  
theta = -arctan(y/sqrt(x<sup>2</sup>+z<sup>2</sup>))  
phi = arctan(z,x)  
{{% /function %}}

{{% function "*Converting a cartesian direction (x,y,z) into a spherical direction (theta,phi) for y2=1:*" %}}  
theta = -y * pi/2  
phi = 0  
{{% /function %}}

[*cos(x)*](http://functions.wolfram.com/ElementaryFunctions/Cos/02) 代表余弦，  
[*sin(x)*](http://functions.wolfram.com/ElementaryFunctions/Sin/02) 代表正弦，  
[*arctan(x)*](http://functions.wolfram.com/ElementaryFunctions/ArcTan/02) 代表反正切，  
[*arctan(x,y)*](http://functions.wolfram.com/ElementaryFunctions/ArcTan2/02) 代表双变量反正切，  
[*sqrt(x)*](http://functions.wolfram.com/ElementaryFunctions/Sqrt/02) 代表平方根。  
如果更喜欢使用直角坐标的话，可以使用这些公式来进行转换。  
这几条公式也作为定向光功能的数学证明（三角函数使用弧度制）。

---

{{% command %}}  
**Route.InitialViewpoint** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***Value***: An integer defining the initial camera viewpoint mode. The following values are valid:  
*0* : The camera will be placed in the cab. (Default)  
*1* : The camera will be placed in 'Exterior Camera' mode.  
*2* : The camera will be placed in 'Track Camera' mode.  
*3* : The camera will be placed in 'Flyby Camera' mode.  
*4* : The camera will be placed in 'Flyby Zooming Camera' mode.  
{{% /command-arguments %}}

该指令允许自定义游戏开始时的视角。

---

{{% command %}}  
**<font color=#555555>Route.DeveloperID</font>**  
{{% /command %}}

<font color=#555555>该指令只被BVE4等使用，openBVE忽略这一指令。</font>

## <a name="train"></a>■ 6. Train (列车) 命名空间

这个命名空间里的指令指定线路使用的列车。

---

{{% command %}}  
Train.Folder *文件夹名*  
Train.File *文件夹名*  
{{% /command %}}

{{% command-arguments %}}  
***文件夹名***：该线路要使用的列车的文件夹名。   
{{% /command-arguments %}}

---

{{% command %}}  
**Train.Run**(*轨道类型编号*)<font color="gray">.Set</font> *走行音编号*  
**Train.Rail**(*轨道类型编号*)<font color="gray">.Set</font> *走行音编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道类型编号***：一个非负整数，代表一个轨道类型。这通过Structure.Rail设定，通过Track.RailType使用。  
***走行音编号***：一个非负整数，代表这个轨道类型使用的列车走行音。  
{{% /command-arguments %}}

列车开发者会为他的列车设定一些在特定轨道种类上的运行时的声音。使用该指令来为轨道类型分配列车走行音。由于走行音编号是由列车开发者指定的，所以需要和列车开发者合作协调，来确保为正确的轨道类型使用相应的走行音。这里有一个[编号标准]({{< ref "/information/standards/_index.md" >}})，但使用得并不很广泛。

---

{{% command %}}  
**Train.Flange**(*轨道类型编号*)<font color="gray">.Set</font> *轮缘摩擦声编号*  
{{% /command %}}

{{% command-arguments %}}  
***RailTypeIndex***: A non-negative integer representing the rail type as defined via Structure.Rail and used via Track.RailType.  
***FlangeSoundIndex***: A non-negative integer representing the train's flange sound to associate to the rail type.  
{{% /command-arguments %}}

列车开发者会为他的列车设定一些在特定轨道种类上的轮缘摩擦声。使用该指令来为轨道类型分配轮缘摩擦声。由于轮缘摩擦声编号是由列车开发者指定的，所以需要和列车开发者合作协调，来确保为正确的轨道类型使用相应的轮缘摩擦声。这里有一个[编号标准]({{< ref "/information/standards/_index.md" >}})，但使用得并不很广泛。

---

{{% command %}}  
**Train.Timetable**(*时刻表编号*)**.Day**<font color="gray">.Load</font> 文件名  
{{% /command %}}

{{% command-arguments %}}  
***时刻表编号***：一个非负整数，代表这个时刻表图片的编号。  
***文件名***：时刻表图片被照亮的一版的文件名，路径应相对于列车文件夹（<sup>首先</sup>考虑），或相对于**Objects**文件夹（<sup>随后</sup>考虑）。  
{{% /command-arguments %}}

用该指令来加载被照亮的时刻表图片。使用Track.Sta命令可以指定在哪站使用哪张时刻表图片。

---

{{% command %}}  
**Train.Timetable**(*时刻表编号*)**.Night**<font color="gray">.Load</font> 文件名  
{{% /command %}}

{{% command-arguments %}}  
***时刻表编号***：一个非负整数，代表这个时刻表图片的编号。  
***文件名***：时刻表图片暗光照下的一版的文件名，路径应相对于列车文件夹（<sup>首先</sup>考虑），或相对于**Objects**文件夹（<sup>随后</sup>考虑）。  
{{% /command-arguments %}}

用该指令来加载暗光照下的时刻表图片。使用Track.Sta命令可以指定在哪站使用哪张时刻表图片。

---

{{% command %}}  
**Train.Gauge** *毫米数值*  
{{% /command %}}

{{% command-arguments %}}  
***毫米数值***：一个浮点数，代表轨道的轨距。单位是**毫米**（0.001m,0.1cm）。默认值是1435（标准轨）。   
{{% /command-arguments %}}

{{% note %}}

Train.Gauge和Route.Gauge作用相同。

{{% /note %}}

---

{{% command %}}  
**Train.Interval** *间隔<sub>0</sub>*; *间隔<sub>1</sub>*; ...; *间隔<sub>n-1</sub>*  
{{% /command %}}

{{% command-arguments %}}  
***Interval<sub>i</sub>***: A floating-point number representing the time interval between the player's train and the preceding train, measured in **seconds**. The default value is 0.  
{{% /command-arguments %}}

{{% note %}}

Train.Interval 和 Route.RunInterval 是一樣的

{{% /note %}}

---

{{% command %}}  
**Train.Velocity** <font color="blue">*速度*</font>  
{{% /command %}}

{{% command-arguments %}}  
**<font color="blue">*速度*</font>**：该指令可以进一步给AI车设置限速。设定为0表示不给AI车设定额外限速。**默认**单位是**km/h**，默认值是0。   
{{% /command-arguments %}}

该指令设定的是额外限速，具体限速仍然受Track.Limit限制。玩家驾驶的列车不受影响，仍然可以一直加速到Track.Limit。

---

{{% command %}}  
**<font color=#555555>Train.Acceleration</font>**  
{{% /command %}}

<font color=#555555>该指令已过时。OpenBVE忽略该指令。</font>

---

{{% command %}}    
**<font color=#555555>Train.Station</font>**    
{{% /command %}}

<font color=#555555>该指令已过时。OpenBVE忽略该指令。</font>

## <a name="structure"></a>■ 7. Structure (结构) 命名空间

这个命名空间里的指令用来载入在后面的其他指令中要用到的物体模型。总体来说，像Track.Rail、Track.FreeObj之类的命令通过*结构编号*来指定要使用的物体。这个*结构编号*对于这个指令是特定的，所以应当先载入要使用的模型。要使用哪个指令，要使用哪个模型，就把它们载入，不需要的就不用载入。

这个命名空间中的指令基本上都是以下这个结构：

{{% command %}}  
**Structure.类型** (_结构编号_)<font color="gray">.Load</font> *文件名*  
{{% /command %}}

*结构编号*是一个非负整数。*文件名*是一个相对于**Object**文件夹的路径，指向要加载的模型文件。*类型*是以下几种之一：

{{% table %}}

| *类型*: | 简介                                                      |
| ---------- | ------------------------------------------------------------ |
| Ground     | 用于Cycle.Ground和Track.Ground的地面模型。           |
| Rail       | 用于Track.Rail、Track.RailStart和Track.RailType的轨道模型。 |
| WallL      | 用于Track.Wall指令的左侧墙壁模型。                         |
| WallR      | 用于Track.Wall指令的右侧墙壁模型。                        |
| DikeL      | 用于Track.Dike指令的左侧路堤模型。                         |
| DikeR      | 用于Track.Dike指令的右侧路堤模型。                        |
| FormL      | 用于Track.Form指令的站台左边缘地面模型。                 |
| FormR      | 用于Track.Form指令的站台右边缘地面模型。                |
| FormCL     | 用于Track.Form指令的可以被变换拉伸的站台左边缘地面模型。<font color="red">不支持ANIMATED格式带动画物体。</font> |
| FormCR     | 用于Track.Form指令的可以被变换拉伸的站台右边缘地面模型。<font color="red">不支持ANIMATED格式带动画物体。</font> |
| RoofL      | 用于Track.Form指令的站台左边缘顶棚模型。                      |
| RoofR      | 用于Track.Form指令的站台右边缘顶棚模型。                     |
| RoofCL     | 用于Track.Form指令的可以被变换拉伸的站台左边缘顶棚模型。 <font color="red">不支持ANIMATED格式带动画物体。</font> |
| RoofCR     | 用于Track.Form指令的可以被变换拉伸的站台右边缘顶棚模型。 <font color="red">不支持ANIMATED格式带动画物体。</font> |
| CrackL     | 用于Track.Crack指令的填充轨道间间隙的可以被变换拉伸的左侧地面模型。<font color="red">不支持ANIMATED格式带动画物体。</font> |
| CrackR     | 用于Track.Crack指令的填充轨道间间隙的可以被变换拉伸的右侧地面模型。 <font color="red">不支持ANIMATED格式带动画物体。</font> |
| FreeObj    | 用于Track.FreeObj指令在轨道旁放置的外景物体模型。                           |
| Beacon     | 用于Track.Beacon指令的轨旁无线电应答器模型。                            |
| 天气     | 为使用 Track.Rain 和 Track.Snow 生成的天气定义对象。 |
| 动态光     | 定义动态照明集。 |

{{% /table %}}

通常支持的物件有 B3D、CSV、X 和 ANIMATED。 但是，FormCL、FormCR、RoofCL、RoofCR、CrackL 和 CrackR 命令只接受 B3D、CSV 和 X 物件。

➟ [关于Forms, Roofs, Cracks的更详尽说明...]({{< ref "/routes/formroofcrack/_index.md" >}})

此外，还有 Structure.Pole 命令，它的语法略有不同：

{{% command %}}  
**Structure.Pole**(_NumberOfAdditionalRails_; _PoleStructureIndex_)<font color="gray">.Load</font> *FileName*  
{{% /command %}}

{{% command-arguments %}}  
***额外轨道跨度值***：一个非负整数，代表这个架线柱模型跨过的额外轨道数量。0代表跨度为1组轨道的架线柱，1代表2组轨道，以此类推。  
***架线柱模型编号***：一个非负整数，代表要加载到的模型编号。  
***文件名***：一个相对于**Object**文件夹的路径，指向要加载的模型文件。  
{{% /command-arguments %}}

特别提示：除FreeObj之外的物体都是在区间块的开始位置被放置的，因此这些模型在建模时尺寸应当保证整个物体头尾的Z坐标伸展区域（放置时朝轨道前方的方向即Z轴正方向）在0~ *区间块长度 *（默认值为25m）之间（这样保证正好盖住整个区间块，否则出现空隙或相互重叠）。在Track命名空间中对应命令处有更详细的解释.

## <a name="texture"></a>■ 8. Texture（材质）命名空间

这个命名空间中的指令指定要使用的天空背景图像和它对齐的方式。

![illustration_background](/images/illustration_background.png)

背景图片被贴在一个环绕着游戏主视角摄像机的圆柱形墙上，从相对于初始前视角方向左偏60°（即十点钟方向）开始，将指定的这张图片重复平铺到Texture.Background(*背景材质编号*).X指令指定的次数（默认为一圈重复六次）。

图片的上方四分之三显示在地平面之上，下方四分之一显示在地平面之下。 通过Texture.Background(*背景材质编号*).Aspect指令，可以选择圆柱体的高度是固定的还是根据图片宽高比自动适应的。如果选择固定高度，圆柱体的高度将是其半径的二分之一，即图片上边缘位于20°视角，下边缘位于-7°视角。如果选择保持图片宽高比，这就不但计算图片宽度和高度，还将参考重复次数计算圆柱高度。

不论需要重复几次，都要保证传入的这张图片在重复时左右边缘可以完美贴合。圆柱的顶面和底面材质是按照图片最上方和最下方的十分之一来计算决定的。因此应尽量避免图片最上方十分之一有高山山峰等物体，否则它们会跑到顶面上去。

除非主轨道位置开头的Track.Back指令应用了一张不同的背景图，Texture.Background(0)的图像会在线路的开头被默认使用。

也可以使用***动态或基于模型的***背景。这个页面详细描述了如何使用这一功能：

[Dynamic & Object Based Backgrounds]({{< ref "/routes/xml/dynamicbackground/_index.md" >}})

---

{{% command %}}  
**Texture.Background**( _背景材质编号_ )<font color="gray">.Load</font> *文件名*  
{{% /command %}}

{{% command-arguments %}}  
***文件名***：一个相对于**Object**文件夹的路径，指向要加载的模型文件或动态XML配置文件。   
{{% /command-arguments %}}

该指令加载用于Track.Back指令的背景图像。

{{% note %}}

如果要使用动态或基于模型的背景，请把它指向需要的XML配置文件。

{{% /note%}}

---

{{% command %}}  
**Texture.Background**( _背景材质编号_ )**.X** *重复次数*  
{{% /command %}}

{{% command-arguments %}}  
***重复次数***：这张图像一圈中重复的次数。默认值是6。   
{{% /command-arguments %}}

{{% note %}}

如使用动态或基于模型的背景，这项设置不起作用。

{{% /note %}}

---

{{% command %}}  
**Texture.Background**( _背景材质编号_ )**.Aspect** *模式*  
{{% /command %}}

{{% command-arguments %}}  
***模式***：宽高比的处理模式。默认值是0（固定高度）。   
{{% /command-arguments %}}

▸ *模式*的可选项：

{{% command-arguments %}}  
**0**：使用固定高度。  
**1**：保留宽高比。  
{{% /command-arguments %}}

{{% note %}}

如使用动态或基于模型的背景，这项设置不起作用。

{{% /note %}}

## <a name="cycle"></a>■ 9. Cycle (循环) 命名空间

{{% command %}}  
**Cycle.Ground(_地面模型编号_)<font color="gray">.Params</font> _地面模型编号<sub>0</sub>_; _地面模型编号<sub>1</sub>_; _地面模型编号<sub>2</sub>_; ...; _地面模型编号<sub>n-1</sub>_**  
{{% /command %}}

{{% command-arguments %}}  
***地面模型编号***：一个非负整数，指定哪一个地面模型编号要被定义为循环的。  
***地面模型编号<sub>i</sub>***：一个非负整数，表示一个先前已经被加载的地面模型是循环的一部分。  
{{% /command-arguments %}}

当平常使用Track.Ground(*地面模型编号*)指令时，选定的模型每个区间块都被重复一次。但是通过Cycle.Ground指令，在使用Track.Ground(*地面模型编号*)时可以把它改成几个不同模型循环出现。这个循环无限重复，*地面模型编号0*从主轨道位置0开始。从技术角度来说，对于一个特定的主轨道位置*p*计算出的*地面模型编号<sub>i</sub>*中的*i*是`Mod[p/区间块长度, n]`（P ÷ 区间块长度 Mod n）。

{{% command %}}  
**Cycle.Rail(_轨道模型编号_)<font color="gray">.Params</font> _轨道模型编号<sub>0</sub>_; _轨道模型编号<sub>1</sub>_; _轨道模型编号<sub>2</sub>_; ...; _轨道模型编号<sub>n-1</sub>_**  
{{% /command %}}

{{% command-arguments %}}  
***轨道模型编号***：一个非负整数，指定哪一个轨道模型编号要被定义为循环的。  
***轨道模型编号<sub>i</sub>***：一个非负整数，表示一个先前已经被加载的轨道模型是循环的一部分。   
{{% /command-arguments %}}

以这一定义为例：

{{% code %}}    
With Structure    
.Ground(0)  grass.csv    
.Ground(1) river.csv    
.Ground(2) asphalt.csv    
{{% /code %}}

这两组代码效果相同：

{{% code %}}    
With Track    
0, .Ground 0    
25, .Ground 1    
50, .Ground 2    
75, .Ground 0    
100, .Ground 1    
125, .Ground 2    
; and so on...    
{{% /code %}}

{{% code "&nbsp;" %}}    
With Cycle    
.Ground(0) 0; 1; 2    
With Track    
0, .Ground 0    
{{% /code %}}

## <a name="signal"></a>■ 10. Signal (信号) 命名空间

这个命名空间内的指令用来加载自定义信号机。

---

{{% command %}}  
__Signal(__ *信号机编号* __)__<font color="gray">.Load</font> *ANIMATED动画模型文件*  
{{% /command %}}

{{% command-arguments %}}  
***信号机编号***：一个非负整数，代表这类信号机的编号。  
***ANIMATED动画模型文件***：一个相对于**Object**文件夹的路径，指向要加载的ANIMATED动画模型文件。  
{{% /command-arguments %}}

用该指令来直接加在一个动画模型文件作为信号机。*信号机编号*可以在使用Track.SigF命令放置这类信号机时使用。

---

{{% command %}}  
__Signal(__ *信号机编号* __)__<font color="gray">.Load</font> *不带扩展名的基座模型文件*; *不带扩展名的发光模型文件*  
{{% /command %}}

{{% command-arguments %}}  
***信号机编号***：一个非负整数，代表这类信号机的编号。  
***不带扩展名的基座模型文件***：一个相对于**Object**文件夹的路径，指向要加载的基座B3D/CSV/……模型文件（不带扩展名）。**必须指定。**  
***不带扩展名的发光模型文件***：一个相对于**Object**文件夹的路径，指向要加载的发光B3D/CSV/……模型文件（不带扩展名）。如果不需要发光效果，可以不指定。  
{{% /command-arguments %}}

该指令加载的信号机切换是通过给同一个模型糊上不同的材质实现的。openBVE按顺序搜索X、CSV、B3D……模型。材质的名字需要是“[模型名字不带扩展名]+[状态编号]+.[扩展名]”这样一个格式。*信号机编号*可以在使用Track.SigF命令放置这类信号机时使用。

举个例子来说，对于*不带扩展名的基座模型文件*，应该有大概这样的一些文件：

_基座模型文件_**.csv**  
_基座模型文件_**<font color="red">0</font>.bmp**  
_基座模型文件_**<font color="red">1</font>.bmp**  
_基座模型文件_**<font color="red">2</font>.bmp**  
_基座模型文件_**<font color="red">n</font>.bmp**

从0到*n*的这一串状态表示中，数值越大表示前方空闲区间越多，允许速度越快。0代表红灯。举个例子来说，内置的信号机是这样规定的：0 (<font color="#C00000">●</font>), 1 (<font color="#FFC000">●●</font>), 2 (<font color="#FFC000">●</font>), 3 (<font color="#00C000">●</font><font color="#FFC000">●</font>), 4 (<font color="#00C000">●</font>) 还有 5 (<font color="#00C000">●●</font>)。可以按照线路信号系统的实际情况增减设置。

文件里的材质指定指令会被忽略，所有面都会被糊上这个同样的状态材质。这表示不能在这个模型里使用一个以上的材质文件（甚至都不能指定），只需要正确地设置材质顶点坐标即可。对于发光模型也是一样的规则。发光模型一般来说是放在信号机前面的一个大方块（虽说也可以用其他的形状，但是由于一般大家都使用透明材质，所以这并没有什么用）。

发光模型材质的处理方法真的值得特别注意。发光模型材质是按照以下的步骤预处理的：

{{% table %}}

| A                                                       | B                                                       | C                                                       | D                                                       | E                                                       | F                                                       |
| ------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| ![illustration_glow_1](/images/illustration_glow_1.png) | ![illustration_glow_2](/images/illustration_glow_2.png) | ![illustration_glow_3](/images/illustration_glow_3.png) | ![illustration_glow_4](/images/illustration_glow_4.png) | ![illustration_glow_5](/images/illustration_glow_5.png) | ![illustration_glow_6](/images/illustration_glow_6.png) |

{{% /table %}}

用作发光材质的图像一般来说应该是椭圆的并且应该有一个锐利的边缘。图形的中心应该完全饱和，并在其外缘加入纯白色。图形的外边既可以是纯黑 (A) 也可以是纯白 (B)。

当openBVE加载这个发光材质时，它会把所有纯黑颜色都换成纯白颜色，变成 (B) 的样子。之后，图像会被反色 (C)，然后进行180°的色调偏移 (D) 。比起 (B) 来说，这使整幅图像的亮度反转，即满饱和的颜色不会被改变（例如中心），亮颜色（例如图形外框）将会变暗，反之亦然。然后，将会进行亮度校正使暗部更暗 (E)，最后再稍稍加上一点模糊效果 (F)。

结果所得到的材质是被叠加混合的。这意味着不是直接在屏幕上覆盖性地绘制材质，而是将材质的颜色以RGB通道值相加的方法叠加在屏幕像素上。加上黑色不会改变原有颜色，而加上完全饱和的颜色就会覆盖原有颜色。例如，加上白色就会得到白色。在设计材质时，请牢记这个处理过程，并遵循它的逆规则。一定要设计出 (A) 或 (B) 这样的图像。

## <a name="track"></a>■ 11. Track (轨道) 命名空间

这个命名空间中的指令定义了轨道的布局。这个命名空间内的指令应该出现在所有其他指令之后，一般位于线路文件的最后部分，且代码量基本都要远超其他部分。

{{% notice %}}

#### 使用主轨道位置

Track命名空间内的所有指令都需要和一个主轨道位置联系起来，这表示该指令执行的位置。一旦出现一个主轨道位置表达式，它后面的Track指令都会在这个位置执行，直到一个新的主轨道位置表达式指定一个新的位置。在主轨道位置表达式出现前写出的Track指令默认会在0位置执行。由于指令会被自动地排序和执行，所以并不一定需要把主轨道位置按照固定顺序写（虽然按照顺序写可以使结构更加清晰）。虽然主轨道位置可以是任意非负浮点数，有一些指令是 **只可以在区间块开始位置（默认为25m的整倍数位置）** 被使用的。默认情况下这意味着它们必须被放在0、25、50、75、100、125、以此类推的位置上。下面有这个特别限制的指令都会被标注出来。

{{% /notice %}}

##### <a name="track_rails"></a>● 11.1. 轨道

---

{{% command %}}  
**Track.RailStart** *轨道编号*; <font color="green">*X*</font>; <font color="green">*Y*</font>; *轨道类型*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个正整数(**>=1**)表示这个编号的轨道要被修改。  
**<font color="green">*X*</font>**：一个浮点数，表示该轨道和主轨道之间的水平距离。**默认的**单位是**米**。正值代表向右偏移，负值代表向左偏移。  
**<font color="green">*Y*</font>**：一个浮点数，表示该轨道和主轨道之间的垂直距离。**默认的**单位是**米**。正值代表向上偏移，负值代表向下偏移。  
***轨道类型***：一个非负整数，表示一个由Structure.Rail或Structure.Cycle指令先前已经载入的模型编号。  
{{% /command-arguments %}}

该指令开始一条由*轨道编号*代表的新轨道。当使用该指令时，*轨道编号*必须是没有被占用的（没有使用过或已经被Track.RailEnd指令声明结束）。*X*、*Y*和*轨道类型*的默认值是0，除非这个*编号*已被占用（不应该这样做）的情况下维持该轨道上次的原值。该指令只用于开始轨道。如果需要更新它的位置，应当使用Track.Rail指令。如果需要结束它，使用Track.RailEnd指令。 在同一位置先终止*一条轨道*再开始同样*编号*的新轨道是允许的。每个区间块中都会根据*轨道类型*自动放置一个对应的轨道模型。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

---

{{% command %}}  
**Track.Rail** *轨道编号*; <font color="green">*X*</font>; <font color="green">*Y*</font>; *轨道类型*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个正整数(**>=1**)表示这个编号的轨道要被修改。  
**<font color="green">*X*</font>**：一个浮点数，表示该轨道和主轨道之间的水平距离。**默认的**单位是**米**。正值代表向右偏移，负值代表向左偏移。  
**<font color="green">*Y*</font>**：一个浮点数，表示该轨道和主轨道之间的垂直距离。**默认的**单位是**米**。正值代表向上偏移，负值代表向下偏移。  
***轨道类型***：一个非负整数，表示一个由Structure.Rail或Structure.Cycle指令先前已经载入的模型编号。  
{{% /command-arguments %}}

该指令开始一条轨道或更新一条轨道的位置或类型。*轨道编号*代表要执行操作的轨道。在该*轨道编号*已存在轨道的情况下*X*,*Y*,*轨道类型*默认维持该轨道上次的原值，否则若未给出，默认值为0。如果需要结束它，使用Track.RailEnd指令。在同一位置先终止*一条轨道*再开始*同样编号*的新轨道是允许的。在每一个区间块中，如果没有Track.Rail指令执行更新，*X*和*Y*值都会保持上一个区间块的同样值。所以，更新*X*或*Y*值只会影响前方区间块的轨道形态。改变*轨道类型*将会改变从指令生效位置开始的轨道类型。如果试图在同一位置反复对同一条轨道进行多次修改，只有第一条指令会起作用，之后的将会被全部忽略。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

{{% warning-nontitle %}}

在该指令上使用为0的轨道编号是无效的，请改用RailType: [勘误备注](https://github.com/leezer3/OpenBVE/wiki/Errata#rail-and-railend-commands-with-an-index-of-zero)

{{% /warning-nontitle %}}

{{% notice %}}

#### 对比Track.RailStart和Track.Rail

这两个都可以用来开始一条轨道。但是当使用Track.RailStart时，openBVE将会检查这个*编号*是否已被占用。所以，使用Track.RailStart来开始轨道是更为明确的，且可以避免由于失误造成的bug。译注：可惜很无奈Hmmsim并不支持Track.RailStart，所以要用Hmmsim的话没得选。

{{% /notice %}}

---

{{% command %}}  
**Track.RailType** *轨道编号*; *轨道类型*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，代表要进行改变的轨道编号。主轨道编号为**0**。默认值是0.  
***轨道类型***：一个非负整数，表示一个由Structure.Rail或Structure.Cycle指令先前已经载入的模型编号。默认值是0。  
{{% /command-arguments %}}

该指令改变由*轨道编号*代表的一条已存在轨道的类型。该轨道必须已经被Track.RailStart或Track.Rail指令创建且还没有被Track.RailEnd指令结束。*轨道类型*的更改将从该指令生效时的轨道位置开始。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}    

---

{{% command %}}  
**Track.RailEnd** *轨道编号*; <font color="green">*X*</font>; <font color="green">*Y*</font>  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个正整数(>=1)表示这个编号的轨道要被结束。  
**<font color="green">*X*</font>**：一个浮点数，表示该轨道和主轨道之间的水平距离。**默认的**单位是**米**。正值代表向右偏移，负值代表向左偏移。  
**<font color="green">*Y*</font>**：一个浮点数，表示该轨道和主轨道之间的垂直距离。**默认的**单位是**米**。正值代表向上偏移，负值代表向下偏移。  
{{% /command-arguments %}}

该指令结束由*轨道编号*代表的一条已存在轨道。*X*和*Y*的默认值是该轨道上次的原值。一旦该指令执行，*该轨道*就会在这一点结束，并在之后视为不再存在。轨道模型不会再继续被放置。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

{{% warning-nontitle %}}

在该指令上使用为0的轨道编号是无效的，请改用RailType: [勘误备注](https://github.com/leezer3/OpenBVE/wiki/Errata#rail-and-railend-commands-with-an-index-of-zero)

{{% /warning-nontitle %}} 

{{% code "*Example of Track.RailStart, Track.Rail, Track.RailType and Track.RailEnd commands*" %}}    
With Track    
1000, .RailStart 1; 3.8; 0.0; 0    
1025, .RailType 1; 1    
1050, .Rail 1; 1.9; 0.0; 0    
1075, .RailEnd 1    
{{% /code %}}

在这个例子中：1号轨道在1000米位置位于主轨道右侧3.8米处开始，使用类型0（这个线路中加载为直线轨道）；在1025米处轨道模型被换为1（加载的一个向左的S型曲线，让它在25米内左偏1.9米，以和下面轨道位置的改变正好对接）；在1050米处轨道被内移至主轨道右侧1.9米处，正好和先前的S型曲线对上，并重新换回直道类型；轨道在1075米处终止，并且在终止时仍旧在主轨道右侧1.9米处。

---

{{% command %}}  
**Track.Accuracy** *精度系数*  
{{% /command %}}

{{% command-arguments %}}  
***精度系数***：一个非负浮点数，表示轨道的精度（质量、平稳度）。默认值是2.   
{{% /command-arguments %}}

该指令设定之后轨道的精度，影响晃动效果和一些物理计算。精度系数需要在0~4之间，超出这个范围的数会被截取到0或4两个值。0代表理想的轨道（没有任何不平），1代表很好的轨道（高速铁路标准），2代表较好的轨道，3代表一般的轨道，4代表较差的轨道。

---

{{% command %}}  
**Track.Adhesion** *附着系数*  
{{% /command %}}

{{% command-arguments %}}  
***附着系数***：一个非负整数，代表轨道的附着系数（数值越大，能产生越大的静摩擦力，车轮越不易打滑），单位是%。默认值是100。   
{{% /command-arguments %}}

该指令设定之后轨道的附着系数。这会影响车轮打滑等物理计算。作为参考，干燥轨面大约是135，湿滑轨面大约是70，降雪轨面大约是50。如果附着系数是0，意味着轨面完全理想光滑，列车车轮将一直打滑，根本不可能移动。

---

{{% command %}}  
**Track.Rain** *Intensity*; *WeatherType* 
{{% /command %}}

{{% command-arguments %}}  
***Intensity***: A non-negative floating-point number measured in percent representing the intensity of the current rainfall. The default value is 0.  
***WeatherType***: A non-negative integer referencing the weather type to use as defined by Structure.Weather from this point onwards.
{{% /command-arguments %}}

This command sets the intensity of the current rainfall, and the weather object to be shown.

{{% warning-nontitle %}}

It is also possible to set the rain intensity using the legacy BVE4 beacon based method. If these commands are present in the route, all Rain commands will be ignored.

{{% /warning-nontitle %}} 

---

{{% command %}}  
**Track.Snow** *Intensity*; *WeatherType* 
{{% /command %}}

{{% command-arguments %}}  
***Intensity***: A non-negative floating-point number measured in percent representing the intensity of the current snowfall. The default value is 0.  
***WeatherType***: A non-negative integer referencing the weather type to use as defined by Structure.Weather from this point onwards.
{{% /command-arguments %}}

This command sets the intensity of the current snowfall, and the weather object to be shown.

{{% notice %}}

#### Combining rain and snow

Rain and snow may be combined, and the simulation will show an appropriate random raindrop or snowflake on the windscreen, based upon the following probabilities:

The base probability for a drop / flake to appear is the greater of the rainfall and snowfall intensities.<br>
The snowfall intensity is then used as the probability that this is a snowflake. <br>
For example, with rainfall at 50 and snowfall at 40, there is a 50% chance for a drop to be added to the windscreen on each tick. <br>
Of this, there is a 40% chance that this will be a snowflake.

{{% /notice %}}

##### <a name="track_geometry"></a>● 11.2. 几何变换

---

{{% command %}}  
**Track.Pitch** *坡度*  
{{% /command %}}

{{% command-arguments %}}  
***坡度***：一个浮点数，单位是‰(千分之一)，代表轨道的坡度。默认值是0。   
{{% /command-arguments %}}

该命令改变之后**所有**轨道的坡度。正值代表上坡，负值代表下坡。0代表平路。*坡度*可以根据在*水平距离*米内上升（下降为负）*垂直高度*米来计算：

{{% function "*Rate expressed through X and Y:*" %}}    
_坡度 = 1000 * 垂直高度 / 水平距离_    
{{% /function %}}

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}    

---

{{% command %}}  
**Track.Curve** <font color="green">*半径*</font>; *超高*  
{{% /command %}}

{{% command-arguments %}}  
**<font color="green">*半径*</font>**：一个浮点数，表示转弯的半径。**默认的**单位是**米**。默认值是0。  
***超高***：一个浮点数，代表弯道的超高（外侧轨道比内侧轨道高出的高度，用来提供向心力），单位**一定**是**毫米**（千分之一米）。默认值是0。另参见Options.CantBehavior。  
{{% /command-arguments %}}

该指令使所有轨道从这一位置开始转弯，并指定转弯的*半径*。正半径代表向右转弯，负半径代表向左转弯。0代表直道。*超高*以毫米指定弯道的超高。对于该数值正负值的含义，请参见Options.CantBehavior。如果它是0(默认)，*超高*的设定值将被取绝对值，并向弯道内侧倾斜，同时在直道上不采取超高；如果它是1，*超高*的符号将被考虑，正数向内倾斜，负数向外倾斜，同时在直道上也会采取超高，负值向左，正值向右。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}} 

---

{{% command %}}  
**Track.Turn** *斜率*  
{{% /command %}}

{{% command-arguments %}}  
***Ratio***: A floating-point number representing a turn. The default value is 0.  
{{% /command-arguments %}}

该指令使轨道在指令插入位置做一个弯折。*斜率*可以根据在*纵向距离*米转向*横向距离*米来计算：

{{% function %}}    
*斜率 = 纵向距离 / 横向距离*  
{{% /function %}}

正值代表右转，负值代表左转，0代表直道。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

{{% warning-nontitle %}}

在创建弯道方面，该指令较为难用，且已过时。如要编辑弯道，可使用Track.Curve。

{{% /warning-nontitle %}}    

---

{{% command %}}  
**Track.Height** <font color="green">*离地高度*</font>  
{{% /command %}}

{{% command-arguments %}}  
**<font color="green">*离地高度*</font>**：一个浮点数，表示主轨道在距离地面的高度。正值会将地面向下移动。**默认的**单位是**米**。   
{{% /command-arguments %}}

该指令告诉游戏在特定位置地面应当距离主轨道下方的距离。它影响通过Structure.Ground和Track.Ground载入和选定的地面模型的放置位置。由于默认的位置0代表轨道的上表面（我想您应该并不希望地面被糊在轨道上面而不是衬在下面），您应该把它设置为一个合适的正值。在相邻的Track.Height之间，高度值被平滑细分以使地面过渡平滑。 例如，以下两段代码是等效的：

{{% code "*Example of a Track.Height command interpolated at 25m boundaries:*" %}}    
1000, Track.Height 1    
1075, Track.Height 4    
{{% /code %}}

{{% code "*Example of Track.Height explicitly set each 25m to produce the same result:*" %}}    
1000, Track.Height 1    
1025, Track.Height 2    
1050, Track.Height 3    
1075, Track.Height 4    
{{% /code %}}  
作为参考顺便提一句，有很多轨道模型都是0.3米高的。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

##### <a name="track_objects"></a>● 11.3. 物件

------

{{% command %}}  
**Track.FreeObj** *轨道编号*; *物体模型编号*; *水平位置*; *垂直位置*; *偏转角*; *俯仰角*; *侧倾角*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定这个物体要被放在哪一条轨道旁。默认值是0.  
***外景物体模型编号***：一个非负整数，指定要被放置的物体模型。默认值是0。  
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。  
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。  
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。  
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

该指令在指定轨道旁放置一个“自由”的外景物体。在放置前需要先用*Structure.FreeObj*载入模型。

------

{{% command %}}  
**Track.Wall** *轨道编号*; *方向*; *墙壁模型编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要放置墙壁的轨道。  
***方向***：指示要放置墙壁的方向。  
***墙壁模型编号***：一个非负整数，指定要被放置的墙壁模型。默认值是0。  
{{% /command-arguments %}}

▸ *方向*的可选项：

{{% command-arguments %}}  
**-1**：放置左侧墙壁(WallL)。  
**0**：放置双侧墙壁(WallL+WallR)。  
**1**：放置右侧墙壁(WallR)。  
{{% /command-arguments %}}

该指令开始或更新一条轨道两侧的墙壁。在放置前需要先用*Structure.WallL*以及*Structure.WallR*载入模型。直到使用Track.WallEnd结束墙壁，对应的墙壁模型会在每个区间块的开始处被不停放置。请注意墙壁是和轨道对应的，这意味着如果先前设定了墙壁，在结束轨道然后重新利用这一相同轨道编号时墙壁依然会被继续放置。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.WallEnd** *轨道编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要停止放置墙壁的轨道。   
{{% /command-arguments %}}

该指令停止在一条轨道两侧放置墙壁模型。该指令从它的位置即刻生效。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.Dike** *轨道编号*; *Direction*; *路堤模型编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要放置路堤的轨道。默认值是0。  
***方向***：指示要放置路堤的方向。  
***路堤模型编号***：一个非负整数，指定要被放置的路堤模型。默认值是0。  
{{% /command-arguments %}}

▸ *方向*的可选项：

{{% command-arguments %}}  
**-1**：放置左侧路堤(DikeL)。  
**0**：放置双侧路堤(DikeL+DikeR)。  
**1**：放置右侧路堤(DikeR)。  
{{% /command-arguments %}}

该指令开始或更新一条轨道两侧的路堤。在放置前需要先用*Structure.DikeL*以及*Structure.DikeR*载入模型。直到使用Track.DikeEnd结束路堤，对应的路堤模型会在每个区间块的开始处被不停放置。请注意路堤是和轨道对应的，这意味着如果先前设定了路堤，在结束轨道然后重新利用这一相同轨道编号时路堤依然会被继续放置。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.DikeEnd** *轨道编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要停止放置路堤的轨道。   
{{% /command-arguments %}}

该指令停止在一条轨道两侧放置路堤模型。该指令从它的位置即刻生效。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.Pole** *轨道编号*; *额外轨道跨度值*; *位置*; *间隔*; *架线柱模型编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要放置架线柱的轨道。默认值是0。  
***额外轨道跨度值***：一个非负整数，代表这个架线柱模型跨过的额外轨道数量。（即这个轨道，再加上 *额外轨道跨度值* 条轨道）0代表跨度为1组轨道的架线柱，1代表2组轨道，以此类推。默认值是0。  
***位置***：指定以3.8米的倍数的放置位置的水平位置（详情见下）。当*额外轨道跨度值*为0时指定放置的方向。  
***间隔***：一个*为区间块长度的倍数*的整数，指定每两个架线柱之间的距离。  
***架线柱模型编号***：一个非负整数，指定要被放置的架线柱模型。默认值是0。  
{{% /command-arguments %}}

该指令开始或更新一条轨道两侧的架线柱。在放置前需要先用Structure.Pole载入模型。架线柱被放在每一个轨道位置是*间隔*的**倍数**的区间块开始位置（这就是为什么可能发现指令的执行位置处并没有一个架线柱）。如果*额外轨道跨度值*是0，*位置*指定放置的方向。如果*位置*小于或等于0，模型被按原样放置（对应左侧架线柱）。如果*位置*大于0，模型会被在X轴向反转180°后放置（对应右侧架线柱）。如果*额外轨道跨度值*大于等于1，*位置*指定以3.8米的倍数的放置位置的水平位置。请注意架线柱是和轨道对应的，这意味着如果先前设定了架线柱，在结束轨道然后重新利用这一相同轨道编号时架线柱依然会被继续放置。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.PoleEnd** *轨道编号*  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号***：一个非负整数，指定要停止放置架线柱的轨道。默认值是0。   
{{% /command-arguments %}}

该指令停止在一条轨道两侧放置架线柱模型。该指令从它的位置即刻生效。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}}  
<font color="gray">**Track.Crack** *轨道编号<sub>1</sub>*; *轨道编号<sub>2</sub>*; *填充间隙地面模型编号*</font>  
{{% /command %}}

该指令将一个物件缩放拉伸，以填补轨道之间的空隙。

{{% command-arguments %}}  
***轨道编号<sub>1</sub>***：一个非负整数，指定第一条轨道。  
***轨道编号<sub>2</sub>***：一个非负整数，指定另一条轨道。  
***填充间隙地面模型编号***：一个非负整数，指定要用来填充间隙的模型(Structure.Crack)。默认值是0。  
{{% /command-arguments %}}

**注意:**  
如果 *轨道编号<sub>1</sub>* 处于 *轨道编号<sub>2</sub>* 的**左侧**（即：其X坐标更小），则使用Structure.CrackL定义的物件，否则使用CrackR。

------

{{% command %}}  
**Track.Ground** *地面模型编号*  
{{% /command %}}

{{% command-arguments %}}  
***地面模型编号***：一个非负整数，指定要被放置的地面模型(Structure.Ground/Cycle.Ground)。   
{{% /command-arguments %}}

该指令设定从这一点开始被放置的地面模型。地面模型总是被放在主轨道下方的一段距离（由Track.Height指定）。如果能查找到定义为*地面模型编号*的循环模型(*Cycle.Ground*)，则优先使用；如果没有，则使用普通模型(*Structure.Ground*)。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

##### <a name="track_stations"></a>● 11.4. 车站

------

{{% command %}}  
**Track.Sta** *站名*; *到达时间*; *发车时间*; *停车铃*; *开门方向*; *强制红灯*; *信号系统*; *到达广播*; *停车时间*; *乘车率*; *发车广播*; *时刻表编号*  
{{% /command %}}

{{% command-arguments %}}  
***站名***：该车站的站名。将会在时刻表和提示信息中显示，所以不应留空。  
***到达时间***：玩家驾驶的列车应当到达此站的[时间]({{< ref "/information/numberformats/_index.md#times" >}})。可以使用特殊值表示额外信息——见下。  
***发车时间***：玩家驾驶的列车应当从此站发车的[时间]({{< ref "/information/numberformats/_index.md#times" >}})。可以使用特殊值表示额外信息——见下。  
***停车铃***：指示停车警铃设备（如果列车有安装）是否应该鸣响以提示列车驾驶在这站停车。默认值是0。  
***开门方向***：指示列车应开启的车门方向。默认值是0。  
***强制红灯***：指示是否应该将最后一个停车点后方的信号机（发车信号机）在玩家驾驶列车接近时设为红色。默认值是0。  
***信号系统***：指示从此站到下一站之间的区间安装的列车信号系统。默认值是0。  
***到达广播***：一个相对于**Sound**文件夹的路径，指向列车停站时播放的音频。  
***停车时间***：一个正浮点数表示列车在车站的最小停车时间（包括开关门时间）。默认值是15。  
***乘车率***：一个非负浮点数表示在该站列车上乘客的相对数量。作为参考，100差不多是正常载客量，而250则表示超载列车。输入值应当在0~250之间。默认值是100。  
***发车广播***：一个相对于**Sound**文件夹的路径，指向在发车前（发车时间-关门时间-音频时长）要播放的音频。  
***时刻表编号***：一个非负整数，指示一个由Train.Timetable(*时刻表编号*)指定的，从此站开始要显示的时刻表。默认保持上站设定。
{{% /command-arguments %}}  

▸ *到达时间*的可选项：

{{% command-arguments %}}  
*time*: The train is expected to arrive at this particular time.  
*omitted*: The train may arrive at any time.  
**P** or **L**: All trains are expected to pass this station.  
**B**: The player's train is expected to pass this station, while all other trains are expected to stop.  
**S**: The player's train is expected to stop at this station, while all other trains are expected to pass.  
**S:**_time_: The player's train is expected to arrive at this particular time, while all other trains are expected to pass.  
**D**: This station is a dummy station for use in conjunction with *ForcedRedSignal*. No stop position overlay or timetable entry will be shown.
{{% /command-arguments %}}

▸ *发车时间*的可选项：

{{% command-arguments %}}  
*[一个时间]*：列车将会在这个时间之后发车。  
*[留空]*：列车只要停足停车时间就可以发车。  
**T**或**=**：这是终点站，列车不会发车。如果*强制红灯*被设定，发车信号机会被无限保持在红灯。  
**T:**_[一个时间]_：这是终点站，列车不会发车。如果*强制红灯*被设定，发车信号机还是会在发车时间后转为绿灯。  
**C**：这是一个“换端传送”车站。详情见下。  
**C:**_[一个时间]_：这是一个“换端传送”车站，除非*停车时间*要求推迟发车，列车会在特定时间传送换端。详情见下。  
**J:**_[车站编号]_：列车可以在该车站"跳转传送"到由*车站编号*指定的车站。详情见下。  
**J:**_[车站编号]**[一个时间]**_：列车可以在该车站"跳转传送"到由*车站编号*指定的车站。除非*停车时间*要求推迟发车，列车将会在指定时间传送。详情见下。  
{{% /command-arguments %}}

▸ *停车铃*的可选项：

{{% command-arguments %}}  
**0**：停车铃不鸣响。  
**1**：停车铃会鸣响来提醒列车驾驶在这站停车。  
{{% /command-arguments %}}

▸ *开门方向*的可选项：

{{% command-arguments %}}  
**L** 或 **-1** ：开左侧门。  
**N** 或 **0** ：不开门。即列车只需要临时停车。  
**R** 或 **1** ：开右侧门。  
**B** ：开两侧门。  
{{% /command-arguments %}}

▸ *强制红灯*的可选项：

{{% command-arguments %}}  
**0**：该车站不影响信号灯。  
**1**：在最后一个停止点后方的信号机（发车信号机）在列车完成停车动作并到达发车时间后才会变绿。  
{{% /command-arguments %}}

▸ *信号系统*的可选项：（由于BVE是日本的模拟游戏所以原装只支持日式信号系统）

{{% command-arguments %}}  
**ATS**或**0**：前方轨道未安装ATC，应切换至ATS。  
**ATC**或**1**：前方轨道已安装ATC，应切换至ATC。  
{{% /command-arguments %}}

该指令设定一个新车站。在编写指令时，该指令应该被放置在车站站台的开始处。其后（到下一站之前）的所有Track.Stop指令都被视为和该站有关。如果该站需要停车，该指令后面应该有一个或多个Track.Stop指令指定车站的停车点，以完成车站的定义。

**特殊功能：**

车站可以被设为“换端车站”。在这样的车站，列车会在满足发车时间后自动被传送到下一站。这个特性可以用来假装列车折返，而不用人工打开菜单跳站。

类似的，车站也可以被设定为”跳站传送“。在这些车站，当到达发车时间，列车就会被传送到指定编号的车站。这样就可以将列车传送到之前的车站，还可以此模拟环线。

{{% warning-nontitle %}}

The first occuring station in a route may not be of the Terminal type.

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.Station** *站名*; *到达时间*; *发车时间*; *强制红灯*; *信号系统*; *发车广播*  
{{% /command %}}

{{% command-arguments %}}  
***站名***：该车站的站名。将会在时刻表和提示信息中显示，所以不应留空。  
***到达时间***：玩家驾驶的列车应当到达此站的[时间]({{< ref "/information/numberformats/_index.md#times" >}})。可以使用特殊值表示额外信息——见下。  
***发车时间***：玩家驾驶的列车应当从此站发车的[时间]({{< ref "/information/numberformats/_index.md#times" >}})。可以使用特殊值表示额外信息——见下。  
***强制红灯***：指示是否应该将最后一个停车点后方的信号机（发车信号机）在玩家驾驶列车接近时设为红色。默认值是0。  
***信号系统***：指示从此站到下一站之间的区间安装的列车信号系统。默认值是0。  
***发车广播***：一个相对于**Sound**文件夹的路径，指向在发车前（发车时间-关门时间-音频时长）要播放的音频。  
{{% /command-arguments %}}

▸ *到达时间*的可选项：

{{% command-arguments %}}  
*[一个时间]*：所有列车都需要停车，玩家驾驶的列车需要在这个特定时间到达。  
*[留空]*：所有列车都需要停车，但可以在任意时间到达。  
**P**或**L**：所有列车都只需要通过这个车站。  
**B**：玩家驾驶的列车只需要通过这个车站，而AI车需要停车。  
**S**：玩家驾驶的列车需要停车，而AI车只需要通过。  
**S:**_[一个时间]_：玩家驾驶的列车需要在这个特定时间到达停车，而AI车只需要通过。  
{{% /command-arguments %}}

▸ *发车时间*的可选项：

{{% command-arguments %}}  
*[一个时间]*：列车将会在这个时间之后发车。  
*[留空]*：列车只要停足停车时间就可以发车。  
**T**或**=**：这是终点站，列车不会发车。如果*强制红灯*被设定，发车信号机会被无限保持在红灯。  
**T:**_[一个时间]_：这是终点站，列车不会发车。如果*强制红灯*被设定，发车信号机还是会在发车时间后转为绿灯。  
**C**：这是一个“换端传送”车站。详情见下。  
**C:**_[一个时间]_：这是一个“换端传送”车站，除非*停车时间*要求推迟发车，列车会在特定时间传送换端。详情见下。  
{{% /command-arguments %}}

▸ *强制红灯*的可选项：

{{% command-arguments %}}  
**0**：该车站不影响信号灯。  
**1**：在最后一个停止点后方的信号机（发车信号机）在列车完成停车动作并到达发车时间后才会变绿。  
{{% /command-arguments %}}

▸ *信号系统*的可选项：（由于BVE是日本的模拟游戏所以原装只支持日式信号系统）

{{% command-arguments %}}  
**ATS**或**0**：前方轨道未安装ATC，应切换至ATS。  
**ATC**或**1**：前方轨道已安装ATC，应切换至ATC。  
{{% /command-arguments %}}

该指令早已过时，请不要使用该指令（它连开门方向都指定不了）——请使用包含更多选项的Track.Sta代替它。  
该指令创建一个新站。对于它没有提供的参数，默认采用以下值：

{{% table-nonheader %}}

| *停车铃*      | 0 （不提醒）                  |
| ---------------- | ----------------------------- |
| *开门方向*          | B （开两侧门） |
| *到达广播*   | 不播放                    |
| *停车时间*   | 15                            |
| *乘车率* | 100                           |
| *时刻表编号* | 不更改                  |

{{% /table-nonheader %}}

该指令设定一个新车站。在编写指令时，该指令应该被放置在车站站台的开始处。其后（到下一站之前）的所有Track.Stop指令都被视为和该站有关。如果该站需要停车，该指令后面应该有一个或多个Track.Stop指令指定车站的停车点，以完成车站的定义。

车站可以被设为“换端车站”。在这样的车站，列车会在满足发车时间后自动被传送到下一站。这个特性可以用来假装列车折返，而不用人工打开菜单跳站。

{{% warning-nontitle %}}

The first occuring station in a route may not be of the Terminal type.

{{% /warning-nontitle %}}

------

{{% command %}}  
**Track.Stop** *标牌方向*; <font color="green">*后方容差*</font>; <font color="green">*前方容差*</font>; *编组数量*  
{{% /command %}}

{{% command-arguments %}}  
***标牌方向***：放置一个默认（日式）停车位置指示牌的方向。  
<font color="green">***后方容差***</font>：一个正浮点数，表示在停止位置后方的容差。**默认的**单位是**米**。默认值是5。  
<font color="green">***前方容差***</font>：一个正浮点数，表示在停止位置前方的容差。**默认的**单位是**米**。默认值是5。  
***编组数量***：一个非负整数，指示该位置适合的列车编组（节数）。0代表该位置适合所有列车编组。默认值是0。  
{{% /command-arguments %}}

▸ *方向*的可选项:

{{% command-arguments %}}  
**-1**：在左侧放置指示牌。  
**0**：不放置默认指示牌。  
**1**：在右侧放置指示牌。  
{{% /command-arguments %}}

This command places a stop point for the last created station. If there is more than one stop defined for a station, the following rules are evaluated in order:
1. If a stop point is defined with the exact number of cars in the train, this will be used.
2. If an all cars stop is defined, this will be used.
3. If no stop points are defined with a matching number of cars, the stop point with the closest greater number of cars will be used.
4. If no stop points have a greater number of cars, the first stop point will be used.

<br>

{{% code "*Example of a station with multiple stop points:*" %}}    
With Track  
0100, .Sta 国际空间站  
0178, .Stop 1;;;4 ,; 适合4编组及以下列车  
0212, .Stop 1;;;6 ,; 适合5~6编组列车  
0246, .Stop 1;;;8 ,; 适合7~8编组列车  
0280, .Stop 1;;;0 ,; 适合9编组及以上列车  
{{% /code %}}

------

{{% command %}}  
<font color="gray">**Track.Form** *轨道编号<sub>1</sub>*; *轨道编号<sub>2</sub>*; *顶棚模型编号*; *站台模型编号*</font>  
{{% /command %}}

{{% command-arguments %}}  
***轨道编号<sub>1</sub>***：要创建站台的轨道。  
***轨道编号<sub>2</sub>***：在创建岛式站台的情况下，输入另一侧轨道的编号；在创建侧式站台的情况下，输入L或R来指定站台创建在轨道左边还是右边。  
***顶棚模型编号***：一个非负整数，指定要被放置的顶棚模型。   
***站台模型编号***：一个非负整数，指定要被放置的站台模型。  
{{% /command-arguments %}}

▸ *轨道编号<sub>2</sub>* 的可选项：

{{% command-arguments %}}    
**任何已开始的轨道**：站台模型将被拉伸变形以形成贴合轨道的岛式站台。   
**L**: 将原样无变形地使用FormL, FormCL 与 RoofL物件。    
**R**: 将原样无变形地使用FormR, FormCR 与 RoofR物件。  
{{% /command-arguments %}}

**注意:**  
如果 *轨道编号<sub>1</sub>* 处于 *轨道编号<sub>2</sub>* 的**左侧**（即：其X坐标更小），则使用Structure.FormL, Structure.FormCL 与 Structure.RoofL定义的物件，否则使用FormR, FormCR与RoofR。


##### <a name="track_signalling"></a>● 11.5. 信号与限速

------

{{% command %}} 
**Track.Limit** <font color="blue">*限速*</font>; *标牌位置*; *方向*   
{{% /command %}} 

{{% command-arguments %}} 
**<font color="blue">*限速*</font>**：一个正浮点数，代表速度。 **默认的** 单位是 **km/h** 。0代表解除限速。默认值是0。   
***标牌位置*** ：放置一个默认日式限速标牌的位置。默认值是0。   
***方向*** ：在道岔限速情况下，标识限速起效的方向。  
{{% /command-arguments %}}

![_限速示意图](/images/illustration_limit.png)

▸ *标牌位置*的可选项：

{{% command-arguments %}}    
**-1**：标牌放在轨道左侧。   
**0**：不放置默认标牌。   
**1**：标牌放在轨道右侧。  
{{% /command-arguments %}}

▸ *方向*的可选项：

{{% command-arguments %}}    
**-1**：该标牌表示通过道岔进入左侧轨道时的限速。   
**0**：该标牌不表示道岔限速。   
**1**：该标牌表示通过道岔进入右侧轨道时的限速。  
{{% /command-arguments %}}

该指令设定从该点开始的速度限制。如果新限速低于原限速，限速将立刻起效；如果新限速高于原限速，限速会在整列列车全部通过标牌之后起效。将*限速*设为`0`将解除限速。将*标牌位置* 设为`-1`或`1`会在轨道的对应一侧放置一个默认的日式限速标牌。将*方向*设为`-1`或`1`会给标牌加上方向显示，用来指示该标牌与道岔方向的关系。*速度*设为`0`的解除限速标牌从来都不会有*方向*显示。 

------

{{% command %}} 
**Track.Section** *状态<sub>0</sub>*; *状态<sub>1</sub>*; *状态<sub>2</sub>*; ...; *状态<sub>n</sub>* 
{{% /command %}} 

{{% command-arguments %}} 
**状态*<sub>i</sub>***：一个非负整数，指示该自动闭塞区段传递给信号机的一个状态。 
{{% /command-arguments %}} 

该指令开始一个自动闭塞区间。这是真正设置自动闭塞系统使其工作的指令，一般和放置显示自动闭塞系统状态的信号机的Track.SigF指令一起使用。*状态<sub>i</sub>*指定区段传递给信号机的一个信号状态。状态0代表不可有列车通过的红灯，越大的值代表前方清空的区间越多，代表列车可以以更快的速度通过。必须指定*状态<sub>0</sub>*。 

{{% notice %}}

#### 默认和简化的两种区间设定方法

openBVE提供两种对*状态<sub>i</sub>*参数的不同解读方式。可以通过Options.SectionBehavior来选择要使用哪种方式。下面简单介绍一下两种方式都是如何定义的。

{{% /notice %}}

**默认解读方式：** 
*状态<sub>i</sub>*指定当前方有i个区间清空（无车）时应该传达给信号机的信号状态。状态被按照书写顺序读入和匹配。但是，如果需要信号机在多种情况下显示相同信息，是可以给多种情况设定相同的信号状态的。

▸ *状态<sub>i</sub>*的意义：

{{% command-arguments %}}    
**状态<sub>0</sub>**：当该区间内已经有列车，或因为停车站ForceRedSignal的缘故被保持在红灯时的信号状态。   
**状态<sub>1</sub>**：当前区间已经清空，但前面的下一个区间有列车时的信号状态。   
**状态<sub>2</sub>**：当前区间已经被清空，下一个区间也已经被清空，但是再下一个区间里就有列车了时的信号状态。   
**状态<sub>n</sub>**：以此类推，当前方有*n*个区间清空时的信号状态。     
{{% /command-arguments %}}

如果前方有比*<sub>n</sub>*个还要多的清空的区间，最后一个状态*<sub>n</sub>*状态会被传给信号机。 

**简化解读方式：**   
*状态<sub>i</sub>*指定这个信号区段会传达给信号机的所有状态。当前方有x个被清空的区间时，被传达给信号机的信号状态就是所有*状态<sub>i</sub>*状态从小到大排的第x个。如果*x*比*状态<sub>i</sub>*状态的总数还要多，最大的一个*状态<sub>i</sub>*状态会被传给信号机。状态的顺序无关紧要，重复出现的值只会被留下一个。 译注：我是没感觉这怎么简化了……

这是Track.Section与Track.SigF一同使用的方法例子：  
{{% code "*Example of a Track.Section command in conjunction with a Track.SigF command:*" %}}    
With Track    
1000, .Section 0;2;4, .SigF 3;0;-3;-1    
{{% /code %}}

------

{{% command %}} 
**Track.SigF** *信号机编号*; *相对关联区间*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角* 
{{% /command %}} 

{{% command-arguments %}} 
***信号机编号***：一个非负整数，指定一个通过Signal(*信号机编号*).Load加载的自定义信号机。   
***相对关联区间***：一个非负整数，指定这个信号机要显示哪一个自动闭塞区间的信号状态。0代表指令执行位置所在的区间，1代表这个区间前方的下一个区间，2代表再前面一个区间，以此类推。   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

该指令放置一个信号机，显示一个由Track.Section指令设定的自动闭塞区间的信号状态。如果将*垂直位置*设为负值（信号机怎么会遁地呢，是不？），这个值会被重设为4.8米，并在后方放置一个默认的架线杆，来代表有个杆的信号机（如果认为默认架线杆太丑或高度有问题，也可以通过在自定义信号机模型里面给它手动加个杆来达到这个效果）。另参见Track.Section。 

在没有自定义相应的Signal(*信号机编号*)的情况下，可以使用这些默认的日式信号机： 

▸ *信号机编号*的默认信号模型：

{{% command-arguments %}}    
**3**：一个三灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●</font>黄和<font color="#00C000">●</font>绿三种状态。   
**4**：一个A型四灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●●</font>双黄、<font color="#FFC000">●</font>黄和<font color="#00C000">●</font>绿四种状态。   
**5**：一个A型五灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●●</font>双黄、<font color="#FFC000">●</font>黄、<font color="#00C000">●</font><font color="#FFC000">●</font>黄绿和<font color="#00C000">●</font>绿五种状态。   
**6**：一个和Track.Relay指令创建的一样的中继信号机。  
{{% /command-arguments %}}

------

{{% command %}}    
**Track.Signal** *状态类型*; *<font color="gray">此处留空</font>*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角*     
**Track.Sig** *状态类型*; *<font color="gray">此处留空</font>*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角*    
{{% /command %}}

{{% command-arguments %}}    
 ***状态类型***：信号机的种类。默认值是2。   
***此处留空***：*BVETs2按照这个参数在信号机下放置一个小文字标签。openBVE并没有这个功能，所以并不使用这个参数。请留空。*   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。   
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

▸ *类型*的可选项：

{{% command-arguments %}}    
[![illustration_signals_small](/images/illustration_signals_small.png)](/images/illustration_signals_large.png)    
**2**：一个A型两灯式信号机，有<font color="#C00000">●</font>红和<font color="#FFC000">●</font>黄两种状态。   
**-2**：一个B型两灯式信号机，有<font color="#C00000">●</font>红和<font color="#00C000">●</font>绿两种状态。   
**3**：一个三灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●</font>黄和<font color="#00C000">●</font>绿三种状态。   
**4**：一个A型四灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●●</font>双黄、<font color="#FFC000">●</font>黄和<font color="#00C000">●</font>绿四种状态。   
**-4**：一个B型四灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●</font>黄、<font color="#00C000">●●</font>黄绿和<font color="#00C000">●</font>绿四种状态。   
**5**：一个A型五灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●●</font>双黄、<font color="#FFC000">●</font>黄、<font color="#00C000">●</font><font color="#FFC000">●</font>黄绿和<font color="#00C000">●</font>绿五种状态。   
**-5**：一个B型五灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●</font>黄、<font color="#00C000">●</font><font color="#FFC000">●</font>黄绿、<font color="#00C000">●</font>绿和<font color="#00C000">●●</font>双绿五种状态。   
**6**：一个六灯式信号机，有<font color="#C00000">●</font>红、<font color="#FFC000">●●</font>双黄、<font color="#FFC000">●</font>黄、<font color="#00C000">●</font><font color="#FFC000">●</font>黄绿、<font color="#00C000">●</font>绿和<font color="#00C000">●●</font>双绿六种状态。  
{{% /command-arguments %}}

该指令在设置自动闭塞区间的同时放置一个信号机。可以指定*状态类型*来选择创建任何一种默认日式信号机。如果将*水平位置*设为0（信号机怎么会搁在轨道正中间呢，是不？），将不会放置信号机，只会设置自动闭塞区间，这和Track.Section的行为有点类似。如果将*水平位置*设为非零，且*垂直位置*设为负值（信号机怎么会遁地呢，是不？），这个值会被重设为4.8米，并在后方放置一个默认的架线杆，来代表有个杆的信号机。 

{{% code "*Example of a four-aspect type B signal without a post at x=-3 and y=5:*" %}}    
1000, Track.Signal -4;;-3;5    
{{% /code %}}

{{% code "*Example of a four-aspect type B signal including a post at x=-3 and y=4.8:*" %}}    
1000, Track.Signal -4;;-3;-1    
{{% /code %}}

Track.Signal和把Track.Section以及Track.SigF一起用有点像。使用Track.Signal更方便，不过，使用Track.Section和Track.SigF可以允许放置自定义信号机，并控制设定更多的参数。

------

{{% command %}} 
**Track.Relay** <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角* 
{{% /command %}} 

{{% command-arguments %}} 
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

该指令创建一个默认的日式中继信号机。这个信号机一般放在弯道等直线能见度不好的地方，显示与前方下一个信号机相同的信号状态，来让列车驾驶提前注意。如果将*水平位置*设为0（信号机怎么会搁在轨道正中间呢，是不？），将不会放置信号机，只会设置自动闭塞区间，这和Track.Section的行为有点类似。如果将*水平位置*设为非零，且*垂直位置*设为负值（信号机怎么会遁地呢，是不？），这个值会被重设为4.8米，并在后方放置一个默认的架线杆，来代表有个杆的信号机。 

##### <a name="track_safety"></a>● 11.6. 车载信号系统

------

{{% command %}} 
**Track.Beacon** *类型*; *轨旁无线电应答器模型*; *相对关联区间*; *数据*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角* 
{{% /command %}} 

{{% command-arguments %}}    
***类型***：一个非负整数，指定这个应答器的类型，会随着数据一起发给车载信号系统。   
***轨旁无线电应答器模型***：一个非负整数，指定一个已经由Structure.Beacon载入的模型。输入-1代表不放置模型。   
***相对关联区间***：一个非负整数，指定这个信号机要显示哪一个自动闭塞区间的信号状态。0代表指令执行位置所在的区间，1代表这个区间前方的下一个区间，2代表再前面一个区间，以此类推。输入-1代表下一个红灯所在的区间。   
***数据***：一个整数，代表特定于要发送给列车车载信号系统的应答器类型的任意数据。译注：openBVE的应答机会在发送信号状态的同时也发送这个附加数据。具体使用就要看ats.dll插件如何编程了。   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

该指令放置一个有源应答器。使用的模型需要通过Structure.Beacon(*BeaconStructureIndex*)预先加载。当列车通过它时，它会把类型、关联区间的信号状态、还有一堆其它信息全部发给车载信号系统接收器。

需要注意，由于Track.Beacon(*类型*)和Track.Transponder(*类型*)是大致相同的，游戏内置的车载信号系统也会接收Track.Beacon的发信。所以，如果要自行编写ats.dll插件，请尽量避开游戏内置车载信号系统已经使用的标号，避免误发信息，造成混乱。详情参见[应答器类型标准]({{< ref "/information/standards/_index.md" >}})。 

------

{{% command %}}    
**Track.Transponder** *类型*; *关联信号机*; *自动系统切换*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角*   
**Track.Tr** *类型*; *关联信号机*; *自动系统切换*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角*  
{{% /command %}}

{{% command-arguments %}}    
***类型***：这个应答器的类型。默认值是0。   
***关联信号机***：这个应答器传递信息的相对来源。默认值是0。   
***自动系统切换***：在安装了两种ATS类型系统的列车上，是否自动切换使用得ATS系统类型。   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

▸ *类型*的可选项：

{{% command-arguments %}}    
![illustration_transponders](/images/illustration_transponders.png)    
**0**：一个ATS-S的S 型发信器，一般放在信号机前600米。   
**1**：一个ATS-S的SN型发信器。一般放在信号机前20米。   
**2**：一个误发车防止型发信器，一般放在车站停站点后。   
**3**：一个ATS-P的更新发信器，一般根据轨道限速等情况放在信号机前600m、280m、180m、130m、85m和50m处。   
**4**：一个ATS-P的停止发信器，一般根据轨道限速等情况放在信号机前25m或30m处。  
{{% /command-arguments %}}

▸ *关联信号机*的可选项：

{{% command-arguments %}}    
**0**：这个应答器传递它前方第一个信号机的状态。   
**1**：这个应答器传递它前方第二个信号机的状态。   
**n**：以此类推，这个应答器传递它前方信号机后面的第*n*个信号机的状态。   
{{% /command-arguments %}}

▸ *自动系统切换*的可选项：

{{% command-arguments %}}    
**-1**：这个应答器并不要求车载信号设备切换模式。   
**0**：当这个应答器的类型是*0*或*1*时，车载信号设备将转到ATS-SN模式；当这个应答器的类型是*3*或*4*时，车载信号设备将转到ATS-P模式。  
{{% /command-arguments %}}

该指令放置一个为内置的ATS-SN或ATS-P信号系统使用的有源应答器。请参见[ATS系统用户指南](http://openbve-project.net/play-japanese/)来了解ATS系统和它的应答器的工作原理。

需要注意，由于Track.Beacon(*类型*)和Track.Transponder(*类型*)是大致相同的，游戏内置的车载信号系统也会接收Track.Beacon的发信。所以，如果要自行编写ats.dll插件，请尽量避开游戏内置车载信号系统已经使用的标号，避免误发信息，造成混乱。详情参见[应答器类型标准]({{< ref "/information/standards/_index.md" >}})。 

➟ [此处了解更多关于ATS-SN和ATS-P的信息](https://openbve-project.net/play-japanese/#3-ats-sn)

➟ [这是一篇讲到如何在实际的路线开发过程中使用这五种应答器来正确设置日式ATS-SN与ATS-P信号系统的教程]({{< ref "/routes/tutorial_ats/_index.md" >}})

------

{{% command %}} 
**Track.AtsSn** 
{{% /command %}} 

该指令放置一个ATS-S的S型发信器，关联前方第一个信号机，且自动切换系统。它等同于**Track.Tr 0;0;0**。 

------

{{% command %}} 
**Track.AtsP** 
{{% /command %}} 

该指令放置一个ATS-P的更新发信器，关联前方第一个信号机，且自动切换系统。它等同于**Track.Tr 3;0;0**。 

------

{{% command %}} 
**Track.Pattern** *类型*; <font color="blue">*速度*</font> 
{{% /command %}} 

{{% command-arguments %}}    
***类型***：速度限制的类型。   
**<font color="blue">*速度*</font>**：一个非负浮点数，指定这里的限速。**默认的**单位是**km/h**。  
{{% /command-arguments %}}

▸ *类型*的可选项：

{{% command-arguments %}}    
**0**：点限速（临时限速、道岔限速）。   
**1**：段限速（永久限速、路段限速）。
{{% /command-arguments %}}

该指令为内置的ATS-P系统设定一个限速，使它自动控制速度，防止列车超速。 

当一个点限速(*类型*为0)被插入在要应用的地方，ATS-P会提前了解这个限速的位置，并提前减速列车使列车通过这一点时速度正好在限速之下。一旦列车通过限速点，速度限制就不再有效，列车将会重新加速。 

当一个段限速(*类型*为1)被插入在应用的地方，ATS-P不会提前了解这个限速的位置，只会从这个位置开始减速列车。为了尽量真实，请把限速设定和ATS-P发信器放在一起。段限速则会被ATS-P记忆并保持，并且直到下一个段限速重新设定才会解除。 

------

{{% command %}} 
**Track.PLimit** <font color="blue">*速度*</font> 
{{% /command %}} 

{{% command-arguments %}} 
**<font color="blue">*速度*</font>**：一个非负浮点数，指定这里的段限速。**默认的**单位是**km/h**。 
{{% /command-arguments %}} 

该指令设定一个段限速，等同于**Track.Pattern 1;_速度_**。 

##### <a name="track_misc"></a>● 11.7. 杂项

------

{{% command %}} 
**Track.Back** *背景材质编号* 
{{% /command %}} 

{{% command-arguments %}} 
***背景材质编号***：一个非负整数，指定一个已经通过Texture.Background(*背景材质编号*)载入的背景材质。 
{{% /command-arguments %}} 

该指令指定从当前位置开始要显示的背景材质。 

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}} 
**Track.Fog** <font color="green">*起始近距离*</font>; <font color="green">*终止远距离*</font>; *红色分量*; *绿色分量*; *蓝色分量* 
{{% /command %}} 

{{% command-arguments %}}    
**<font color="green">*起始近距离*</font>**：一个浮点数，指示雾的起始位置距离摄像机的距离。**默认的**单位是**米**。默认值是0。   
**<font color="green">*终止远距离*</font>**：一个浮点数，指示雾的终止位置距离摄像机的距离。**默认的**单位是**米**。默认值是0。   
***红色分量***：一个0~255之间的整数，表示雾颜色的红色值。默认值是128。   
***绿色分量***：一个0~255之间的整数，表示雾颜色的绿色值。默认值是128。   
***蓝色分量***：一个0~255之间的整数，表示雾颜色的蓝色值。默认值是128。  
{{% /command-arguments %}}

该指令可以启用、更新或停用雾效果。如果需要启用雾，*起始近距离*必须小于*终止远距离*。如果需要停用雾，只需将*起始近距离*设得大于等于*终止远距离*。 

雾效果影响物体绘制的颜色。在起始近距离以内的物体颜色保持原样，在终止远距离外的物体颜色和雾颜色完全相同，在两个位置中间的物体颜色是线性平滑渐变的。雾也会影响背景图像。不论实际情况如何，在计算时背景图像都会被假定距离摄像机600米远。 

根据Options.FogBehavior的设定，雾的处理方式会有所不同。详情见上。基于区间块模式下，雾的颜色和范围从原先Track.Fog设置时的区间块的到新Track.Fog设置时的区间块以线性插值平滑过渡；线性插值模式下，雾的颜色和范围从两次Track.Fog调用的位置之间线性插值平滑过渡。这和Track.Brightness的效果类似。

{{% warning-nontitle %}}

该指令只能在区间块开始位置使用。

{{% /warning-nontitle %}}

------

{{% command %}} 
**Track.Brightness** *亮度值* 
{{% /command %}} 

{{% command-arguments %}} 
***亮度值***：一个位于0~255之间的整数。默认值是255。 
{{% /command-arguments %}} 

该指令控制驾驶台的照明亮度情况。*亮度值*按照0（黑暗）到255（明亮）测算，并在连续的Track.Brightness指令之间平滑渐变。只要是会影响驾驶室内的亮度的地方（例如隧道、桥梁、车站顶棚、等等，尤其注意隧道灯和顶棚房梁的处理，虽然十分繁琐，但是能大大提高真实度），都应该使用这条指令来进行微调。 

{{% code "*Example:*" %}}    
With Track   
1200, .Brightness 255 ,; 桥前   
1205, .Brightness 128 ,; 桥下   
1210, .Brightness 255 ,; 桥后  
{{% /code %}}

------

{{% command %}} 
**Track.Marker** *文件*; <font color="green">*持续距离*</font> 
{{% /command %}} 

{{% command-arguments %}} 
***文件***：相对于**Object**文件夹的标识图片路径。   
<font color="green">***持续距离***</font>：一个非零浮点数，代表标识图片持续显示的距离。**默认的**单位是**米**。   
{{% /command-arguments %}}

▸ *持续距离*的意义：

{{% command-arguments %}}    
*负值*：图片从指令执行位置开始显示，并持续显示(－*持续距离*)米。   
*正值*：图片从指令执行位置前*持续距离*米开始显示，并正好在指令执行位置结束。  
{{% /command-arguments %}}

这条指令显示一个所谓的标识图片在屏幕的右上角。可以把它用来传递建议或信息。图片中颜色是 [64,64,64]，即#404040的像素点会被绘制为透明的。 

------

{{% command %}} 
**Track.Marker** *XML文件* 
{{% /command %}} 

也可以使用一个使用XML文件的*Track.Marker*指令。 这提供比起原来的标识图片来说更加复杂可控的行为。 

请参见 [XML标记指南]({{< ref "/routes/xml/route_marker/_index.md" >}})

------

{{% command %}} 
**Track.TextMarker** *文字*; <font color="green">*持续距离*</font>; *字体颜色* 
{{% /command %}} 

{{% command-arguments %}}    
***文字***：要显示的标识文字。（不支持特殊字符）   
<font color="green">***持续距离***</font>：一个非零浮点数，代表标识文字持续显示的距离。**默认的**单位是**米**.   
***字体颜色***：标识文字显示时的字体颜色。  
{{% /command-arguments %}}

▸ *持续距离*的意义：

{{% command-arguments %}}    
*负值*：图片从指令执行位置开始显示，并持续显示(－*持续距离*)米。   
*正值*：图片从指令执行位置前*持续距离*米开始显示，并正好在指令执行位置结束。  
{{% /command-arguments %}}

▸ *字体颜色*的可选项：

{{% command-arguments %}}    
*1*：高端黑。   
*2*：隐喻灰。   
*3*：纯洁白。   
*4*：姨妈红。   
*5*：阳光橙。   
*6*：原谅绿。   
*7*：深邃蓝。   
*8*：少女粉。  
{{% /command-arguments %}}

这条指令创建一个简单的文字标识，和其他信息一样显示在屏幕左上角。

------

{{% command %}}    
**Track.PointOfInterest** *轨道编号*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角*; *文字描述*   
**Track.POI** *轨道编号*; *水平位置*; *垂直位置*; *偏转角*; *俯仰角*; *侧倾角*; *文字描述*
{{% /command %}}

{{% command-arguments %}} 
***轨道编号***：一个非负整数，代表这个兴趣点所在的轨道。   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。  
***偏转角***：相机在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：相机在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：相机在XY平面上转动的角度（相对于后方顺时针）。默认值是0。   
***文字描述***：一行描述这个兴趣点的文字。   
{{% /command-arguments %}}


这条指令创建一个用户可以使用CAMERA_POI_PREVIOUS（小键盘1）或CAMERA_POI_NEXT（小键盘7）跳转的一系列在轨道旁边的可能比较有意思的观察点。摄像机会被按照指定的方向放在指定的位置。如果设置了*文字描述*，在跳转时它会被显示，让玩家更好的了解这里是哪儿，为什么这里有意思。

------

{{% command %}} 
**Track.PreTrain** *通过时间* 
{{% /command %}} 

{{% command-arguments %}} 
***通过时间***：先行列车需要在这个轨道位置的[时间]({{< ref "/information/numberformats/_index.md#times" >}})。 
{{% /command-arguments %}} 

这条指令为一列不可见的列车指定它在固定时间要到达的地点，来影响自动闭塞信号显示。与Route.RunInterval来对比，该指令创建的隐形现行列车只是为了影响信号而已，并且可以指定它到达固定地点的特定时间。先行列车不会开倒车(EZY)，也不会做时空穿越，所以时间和位置必须是升序指定的，即不可以让它在更早的时间到达更远的位置，因为这明显违反了客观的物理定律。在第一个指定的时间之前，这列不可见列车留在起始位置。在第一个和最后一个指定的时间之间，这列不可见列车线性（即没有加减速的曲线变化）地在指定的位置之间移动。在最后一个指定的时间之后，这列不可见列车立刻消失，并清空它影响的所有区间。 

------

{{% command %}} 
**Track.Announce** *文件*; <font color="blue">*参考速度*</font> 
{{% /command %}} 

{{% command-arguments %}}    
***文件***：相对于**Sound**文件夹的要播放的声音的路径。   
<font color="blue">***参考速度***</font>：如果该声音的播放速度会随列车速度改变，输入该声音录制时列车的行驶速度作为参考。如果该声音的播放速度不随列车速度改变，输入0或留空。  
{{% /command-arguments %}}

该指令在玩家驾驶的列车通过该点时在驾驶台中播放一个声音。如果*参考速度*被设为0（默认），声音被原样播放（例如车内广播）。如果给出了*参考速度*，声音在列车速度等于参考速度时原调播放，在其他速度情况下按速度变调（例如自定义的轮缘摩擦声、道岔撞击声）。 

------

{{% command %}} 
**Track.Doppler** *文件*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font> 
{{% /command %}} 

{{% command-arguments %}}    
***文件***：相对于**Sound**文件夹的要播放的声音的路径。   
**<font color="green">*水平位置*</font>**：声源距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。   
**<font color="green">*垂直位置*</font>**：声源距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。默认值是0。 
{{% /command-arguments %}}

这条指令在对应位置放置一个环境声音（例如平交道口警报器）。这个声音会被无限重复播放，并具有真实的多普勒效果。（请注意：openBVE中的所有声音都有多普勒效果） 

------

{{% command %}}    
**Track.Buffer**    
{{%/command %}}

该指令创建一个不可见的缓冲器，用来停住列车。列车可以从从前后双向撞上它。在线路的开始和结束处都放上这个指令，以防止玩家驾车出图。该指令并不会放置可见的物体，所以如果必要，使用Track.FreeObj放置一个可见的实际代表物体。

------

{{% command %}} 
**Track.Destination** *筛选类型*; *轨旁无线电应答器模型*; *正向通过目的地*; *反向通过目的地*; *单次触发*; <font color="green">*水平位置*</font>; <font color="green">*垂直位置*</font>; *偏转角*; *俯仰角*; *侧倾角* 
{{% /command %}} 

{{% command-arguments %}}    
***筛选类型***：筛选这个应答机对于哪些列车有效。*-1*代表只应用于AI列车，*0*代表应用于所有列车，*1*代表只应用于玩家列车。   
***轨旁无线电应答器模型***：一个非负整数，指定一个已经被Structure.Beacon载入的模型。输入-1代表不放置物体。   
***正向通过目的地***：一个整数，代表当列车正向通过时要改变为的目的地显示索引值。*-1*代表不改变。   
***反向通过目的地***：一个整数，代表当列车反向通过时要改变为的目的地显示索引值。*-1*代表不改变。   
***单次触发***：如果设为*0*，这个应答机会向每一列车发送。如果设为*1*，只会发送给通过它的第一列列车。   
**<font color="green">*水平位置*</font>**：物体距离轨道中心的水平距离。**默认的**单位是**米**。正值代表向右，负值代表向左。默认值是0。     
**<font color="green">*垂直位置*</font>**：物体距离轨道中心的垂直距离。**默认的**单位是**米**。正值代表向上，负值代表向下。     
***偏转角***：该物体在XZ平面上转动的角度（相对于上方顺时针）。默认值是0。   
***俯仰角***：该物体在YZ平面上转动的角度（相对于左方顺时针）。默认值是0。   
***侧倾角***：该物体在XY平面上转动的角度（相对于后方顺时针）。默认值是0。  
{{% /command-arguments %}}

这条指令设定一个特殊的无源应答机，设定列车的目的地属性变量，并影响支持这个功能的车载信号系统插件和动画外饰物体。要使用的模型需先通过Structure.Beacon(*BeaconStructureIndex*)载入。

------

{{% command %}}  
**Track.HornBlow** *Type*; *BeaconStructureIndex*; *TriggerOnce*; *<font color="green">X</font>*; *<font color="green">Y</font>*; *Yaw*; *Pitch*; *Roll*  
{{% /command %}}

{{% command-arguments %}}  
***Type***: Defines the type of horn to play: *0* for the primary horn, *1* for the secondary horn and *2* for the music horn.  
***BeaconStructureIndex***: A non-negative integer representing the object to be placed as defined via Structure.Beacon, or -1 to not place any object.  
***TriggerOnce***: If set to *0*, this beacon will be triggered by all valid trains which pass over it. If set to *1*, it will be triggered by the first valid train only.  
***<font color="green">X</font>***: The X-coordinate at which to place the object, **by default** measured in **meters**. The default value is 0.  
***<font color="green">Y</font>***: The Y-coordinate at which to place the object, **by default** measured in **meters**. The default value is 0.  
***Yaw***: The angle in degrees by which the object is rotated in the XZ-plane in clock-wise order when viewed from above. The default value is 0.  
***Pitch***: The angle in degrees by which the object is rotated in the YZ-plane in clock-wise order when viewed from the left. The default value is 0.  
***Roll***: The angle in degrees by which the object is rotated in the XY-plane in clock-wise order when viewed from behind. The default value is 0.  
{{% /command-arguments %}}

This command places a special beacon, which commands an AI driver to play the horn. Both an AI controlled player train and pure AI trains will trigger this beacon, unless **TriggerOnce** is set. The object must have been loaded via Structure.Beacon(*BeaconStructureIndex*) prior to using this command.

------

{{% command %}}  
**Track.DynamicLight** *DynamicLightIndex*  
{{% /command %}}

{{% command-arguments %}}  
***DynamicLightIndex***: A non-negative integer, representing the Dynamic Lighting set to be used from this point onwards, as defined by Structure.DynamicLight
{{% /command-arguments %}}

该指令可以被用来替代*Route.AmbientLight*，*Route.DirectionalLight*和*Route.LightDirection*。

它允许根据时间改变光照，具体细节在这一页上说明：

[Dynamic Lighting]({{< ref "/routes/xml/dynamiclight/_index.md" >}})

---

{{% command %}}  
**Track.AmbientLight** *RedValue*; *GreenValue*; *BlueValue*  
{{% /command %}}

{{% command-arguments %}}  
***红色分量***：一个0~255之间的整数，表示漫射环境光颜色的红色值。默认值是160。  
***绿色分量***：一个0~255之间的整数，表示漫射环境光颜色的绿色值。默认值是160。：一个0~255之间的整数，表示漫射环境光颜色的红色值。默认值是160。  
***蓝色分量***：一个0~255之间的整数，表示漫射环境光颜色的蓝色值。默认值是160。  
{{% /command-arguments %}}

This command defines the ambient light color to be used from this point onwards. All polygons in the scene are illuminated by the ambient light regardless of the light direction.

---

{{% command %}}  
**Track.DirectionalLight** *RedValue*; *GreenValue*; *BlueValue*  
{{% /command %}}

{{% command-arguments %}}  
***红色分量***：一个0~255之间的整数，表示定向照射光颜色的红色值。默认值是160。  
***绿色分量***：一个0~255之间的整数，表示定向照射光颜色的绿色值。默认值是160。  
***蓝色分量***：一个0~255之间的整数，表示定向照射光颜色的蓝色值。默认值是160。  
{{% /command-arguments %}}

This command defines the directional light to be used from this point onwards. The polygons in the scene are only fully illuminated by the directional light if the light direction points at the front faces. If pointing at back faces, no light is contributed. *Route.LightDirection* should be set to specify the light direction.

---

{{% command %}}  
**Track.LightDirection** *Theta*; *Phi*  
{{% /command %}}

{{% command-arguments %}}  
***θ 倾斜角***：一个以**角度**为单位的浮点数，控制定向光的仰角。默认值是60。  
***φ 方位角***：一个以**角度**为单位的浮点数，控制定向光的方位。默认值是-26.57。  
{{% /command-arguments %}}

This command defines the light direction from this point onwards. This is the direction the light shines at, meaning the opposite direction the sun is located at. First, *Theta* determines the pitch. A value of 90 represents a downward direction, while a value of -90 represents an upward direction. With those extremes, the value of *Phi* is irrelevant. A value of 0 for *Theta* represents a forward direction originating at the horizon behind. Once the pitch is defined by *Theta*, *Phi* determines the rotation in the plane. A value of 0 does not rotate, a value of 90 rotates the direction to the right and a value of -90 rotates to the left. A backward direction can be both obtained by setting *Theta* and *Phi* to 180 and 0 or to 0 and 180, respectively. Values in-between allow for more precise control of the exact light direction.

{{% warning-nontitle %}}

The direction of the light is relative to the initial direction at the zero-point of the route (e.g Track position **0**), not the current position.

{{% /warning-nontitle %}}

![__光照方向示意图](/images/illustration_light_direction.png)

将球面坐标(θ,φ)换算为直角坐标(x,y,z)的公式：  
{{% function "*Converting a spherical direction (theta,phi) into a cartesian direction (x,y,z):*" %}}    
x = cos(theta) * sin(phi)    
y = -sin(theta)    
z = cos(theta) * cos(phi)    
{{% /function %}}

{{% function "*Converting a cartesian direction (x,y,z) into a spherical direction (theta,phi) for y2≠1:*" %}}  
theta = -arctan(y/sqrt(x<sup>2</sup>+z<sup>2</sup>))  
phi = arctan(z,x)  
{{% /function %}}

{{% function "*Converting a cartesian direction (x,y,z) into a spherical direction (theta,phi) for y2=1:*" %}}  
theta = -y * pi/2  
phi = 0  
{{% /function %}}

[*cos(x)*](http://functions.wolfram.com/ElementaryFunctions/Cos/02) 代表余弦，  
[*sin(x)*](http://functions.wolfram.com/ElementaryFunctions/Sin/02) 代表正弦，  
[*arctan(x)*](http://functions.wolfram.com/ElementaryFunctions/ArcTan/02) 代表反正切，  
[*arctan(x,y)*](http://functions.wolfram.com/ElementaryFunctions/ArcTan2/02) 代表双变量反正切，  
[*sqrt(x)*](http://functions.wolfram.com/ElementaryFunctions/Sqrt/02) 代表平方根。  
如果更喜欢使用直角坐标的话，可以使用这些公式来进行转换。  
这几条公式也作为定向光功能的数学证明（三角函数使用弧度制）。

---