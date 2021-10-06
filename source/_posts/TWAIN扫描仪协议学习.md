# TWAIN扫描仪协议学习

author: 星光

weixi:xingguangzfs



## （一）TWAIN介绍

TWAIN协议，是一个软件和数码相机、扫描仪等图像输入设备之间的通讯标准。简言之，通过TWAIN协议，软件只需要用一种方法，就可以获得所有支持该协议的图像设备中的图像，不管你是数码相机、扫描仪、还是其它的图像设备。

TWAIN协议创建的目的就是兼容从不同外设上获取静态图像。TWAIN协议为操作系统提供了软件支持，使得符合TWAIN协议的软件通过调用TWAIN协议接口就能从兼容TWAIN协议的外设上获取静态图像，而不必考虑外设的功能差别。

从硬件到软件，TWAIN被设计成四层结构：硬件、源、源管理器和软件。

硬件厂家是通过硬件驱动程序支持TWAIN协议。TWAIN的硬件层接口被称为源，源管理器负责管理来自不同硬件厂家的源。软件不直接调用硬件厂家的TWAIN接口，而是通过向源管理器发送命令方式间接操控硬件。



## （二）TWAIN结构

TWAIN依靠三个组件协同完成与图像设备间的通讯和数据传输，这三个组件就是：**Application**，**Source Manager**和**Source**。

（1）**Application**：就是你所要编写的应用程序。

（2）**Source Manager**：由TWAIN提供的源管理器，它会搜集本地安装的所有符合TWAIN标准的图像设备，并根据需要去加载。同时担任Application和Source之间的桥梁。实际上Application 对Source的所有操作都是由Source Manager代劳的。

（3）**Source**：指代图像设备，实际在系统中是指由设备供应商提供的支持TWAIN协议的DLL文件，Source Manager通过调用里面的方法与设备进行通讯和数据传输。



```
在实际应用中，Application与Source Manager间的通讯是通过TWAIN提供的DSM_Entry()函数实现。

而Source Manager而与Source之间的通讯是通过TWAIN提供的DS_Entry()函数实现。
```



## （三）TWAIN接口函数

要编写应用程序实现与支持TWAIN协议的图像设备通讯，只需要了解上面提到DSM_Entry()接口函数就够了。这个过程有点类似于HTTP的一问一答模式，通过DSM_Entry()向Source Manager发送一条命令，就可以控制图像设备，或者获取设备信息。内部Source Manager会帮忙处理好中间的过程，应用程序无需关心。

TWAIN定义了一百多个命令消息，Application只需要通过DSM_Entry()函数将命令消息发送给Source Manager，就可以控制图像设备。TWAIN把它定义的操作称为Triplets操作，每个操作用三个定义参数来表示，而这三个参数用不同的前缀来区分。每个Triplets操作都是唯一的，不会有歧意，它代表一个特定的操作行为。这三个前缀分别是：

Data Group(前缀名DG<u>_</u>)、Data Argument(前缀名DAT<u>_</u>)、Message ID(前缀名MSG<u>_</u>)，每个参数都有各自包含的信息。比如：DG_CONTROL/DAT_PARENT/MSG_OPENDSM代表打开Source Manager管理器，而DG_CONTROL/DAT_PARENT/MSG_CLOSEDSM则表示关闭已经打开的Source Manager。更多的定义可以查询相关手册资料，或者见后面附录。

现在我们知道了DSM_Entry()函数，那么怎么得到它呢?就需要在你的应用程序中包含twain.h头文件，并且加载相应的库，这些库在不同的平台由不同的文件实现，比如windows下调用LoadLibrary加载twain_32.dll/twain_64.dll库，并调用GetProcAddress获取DSM_Entry()函数的地址指针，之后就可以调用DSM_Entry向设备发送指令了。

现在我们已经明白TWAIN所有的操作都是通过DSM_Entry（）函数来实现的，我们先来持下它的定义：

```
DSMENTRY DSM_Entry(TW_IDENTITY  *_pOrigin, // 指示发起者指针
                   TW_IDENTITY  *_pDest, // 指示接收者指针
                   TW_UINT32     _DG, // Triplets操作DG参数
                   TW_UINT16     _DAT, // Triplets操作DAT参数
                   TW_UINT16     _MSG, // Triplets操作MSG参数
                   TW_MEMREF     _pData)// 返回数据块的指针
```

函数执行成功，会返回TWRC_SUCCESS，失败会返回错误码，具体的可以参数手册或者后面的附录。



## （四）TWAIN处理流程



Application、Source Manager、Source之间要实现数据传输，必须遵循一定的流程，不同的操作需要放到合适的流程中去处理。为此，TWAIN规定了S1到S7共**7种状态**来管理不同的流程，这7种状态分为两类，S1到S3为Source Manager所有，而S4到S7为Source所有，各状态如下所示：

**状态1**：准备会话，此时Application与Source Manager还没有建立连接，此时标志位为1；

**状态2**：加载Source Manager，此时已经成功加载到内存中，但是还没有打开，已经做好了接受Triplets操作请求的准备，此时标志位为2；

**状态3**：打开Source Manager，此时已经成功打开，做好了管理source的准备，此时标志位为3；

**备注**：一个Application只能打开一个Source Manager，而一个Source Manager可以管理多个Source，并且多个Source之间互相独立，互不干扰。

**状态4**：打开Source，此时可以查询或设置Source的属性，比如分辨率，是否支持彩色等，此时标志位为4；

**状态5**：Source可用，此时Source已经打开成功，并初始化完成，可以正常工作了。此时可以发一个T riplets操作，选择是否让Source显示它自身的操作界面，此时标志位为5；

状态**6**：准备数据传输，此时Source已经准备好为Application传输数据，如果有音频数据要传输，则Application必须确保在传输图像数据之前，先把音频数据传输完成，此时标志位为6；

**状态7**：数据传输开始，Source开始数据传输，此时标志位为7；



**下面我将依照操作流程简要介绍状态转移过程：**

（1）程序刚启动时，默认状态是S1；



（2）初始化库，获取到DSM_Entry()函数指针后，状态会从S1转到S2；



（3）打开Source Manager成功后，状态从S2转到S3；

​	Triplets操作：DG_CONTROL/DAT_PARENT/MSG_OPENDSM

​	**S3状态下可以执行的主要操作有：**

​	DG_CONTROL/DAT_IDENTITY/MSG_USERSELECT ： 显示系统默认的设备选择列表；

​	DG_CONTROL/DAT_IDENTITY/MSG_GETDEFAULT ：获取默认图像设备；

​	DG_CONTROL/DAT_IDENTITY/MSG_GETFIRST ：遍历图像列表时使用，获取列表第一个设备；

​	DG_CONTROL/DAT_IDENTITY/MSG_GETNEXT ： 遍历图像列表时使用，获取列表下一个设备；

​	返回操作  : 执行DG_CONTROL/DAT_PARENT/MSG_CLOSEDSM操作，则状态会从S3转到S2；

​	

（4）打开某个Source后，状态从S3转到S4，S4状态下可以设置或获取源的各种性能参数；

​	Triplets操作：DG_CONTROL/DAT_IDENTITY/MSG_OPENDS

​	**S4状态下可执行的主要操作：**

​	DG_CONTROL/DAT_CAPABILITY/MSG_GET : 获取源的属性

​	DG_CONTROL/DAT_CAPABILITY/MSG_SET : 设置源的属性

​	通过上述两个操作，可以查询设备的信息，或者定制设备的功能。



（5）请求从Source获取数据，状态从S4转到S5，比如对于扫描仪，就是请求扫描；

​	Triplets操作：DG_CONTROL/DAT_USERINTERFACE/MSG_ENABLEDS



（6）当准备传输数据时，状态从S5转到S6；

​	Triplets操作：DG_CONTROL/DAT_EVENT/MSG_PROCESSEVENT

​	**备注**：MSG_PROCESSEVENT是由Source发送的，而不是Application发送，Source Manager需要	监听MSG_PROCESSEVENT事件，并且当为MSG_XFERREADY时表示数据已经准备好，状态就从S5	转到S6；



（7）开始进行数据传输，状态从S6转到S7；

​	Triplets操作：DG_IMAGE/DAT_IMAGEINFO/MSG_GET或

​							DG_IMAGE/DAT_IMAGENATIVEXFER/MSG_GET等



（8）终止数据传输，状态从S7转到S6，再转到S5；

​	Triplets操作：DG_CONTROL/DAT_PENDINGXFERS/MSG_ENDXFER



（9）断开会话，即取消某个会话，状态从S5转到S4；

​	Triplets操作：DG_CONTROL/DAT_USERINTERFACE/MSG_DISABLEDS



（10）关闭会话，即关闭某个Source，状态从S4转到S3；

​	Triplets操作：DG_CONTROL/DAT_USERINTERFACE/MSG_CLOSEDS



（11）关闭Source Manager，状态从S3转到S2；

​	Triplets操作：DG_CONTROL/DAT_PARENT/MSG_CLOSEDSM



## （五）TWAIN数据传输模式

TWAIN协议定义了三种数据传输模式，用于把数据从Source传输到Application。分别是本地模式、文件模式和缓存模式。下面对每一种模式做简单介绍：

**备注**：对于音频数据，只能选择本地模式或文件模式进行传输。

**（1）本地模式：**

所有图像设备都支持本地模式，并且是TWAIN默认的传输模式，它是最容易实现的数据传输模式，但它有一定局限性，只支持DIB图像数据，并且传输时会受到内存大小的限制。

**实现原理**：Source在传输数据时，会单独分配一块内存，把数据存储在此内存中，然后把内存地址告诉Application，Application收到通知后，从内存中获取图像数据，并且要负责释放内在。此模式下，如果当前系统可用内存小于图像大小，则会导致传输失败。

**（2）文件模式**

该模式是让Application创建一个文件，这个文件用于存储传输的数据，Source将对此文件进行读写操作。Source把要传输的数据写入此文件中，Application通过访问该文件，就可获取传输的数据。

当使用本地模式传输一个大的图像数据，内在不够大时，可以使用该模式来传输。文件模式与缓存模式相比，操作简单，但是传输速度要慢，并且传输完成后，Application还需要去管理此文件。

**（3）缓存模式**

缓存模式在整个传输过程中，使用一个或多个内在缓存区，内存的分配和释放由Application控制。传输过程中，数据被当成一个字节流进行处理，Application必须使用TW_IMAGEINFO 和 TW_IMAGEMEMXFER操作，去获取各个缓存区的信息并把它们正确组织为一个完整的位图。

如果使用本地模式或者文件模式去处理，则整个传输过程只需要一个Triplets操作就可以完成，但缓存模式，必须要发送多个Triplets操作，不断去获取缓存冲数据，直到获取完成为止。但是，该模式具有很好的灵活性，可以更好的控制数据的获取方式，只是编程上会麻烦一些。



## （六）TWAIN应用实践

经过以上的学习，相信大家对TWAIN协议有了一定的了解，学习的目的是为了应用，接下来最重要的就是应用以上学习的知识，来实现一个自己的扫描仪控制应用程序。很庆幸的是在github上找到了一个非常不错的开源项目，利用这个项目，可以很方便、很快速的实现自己的扫描工具。

TWAIN下载地址：https://github.com/twain

**（1）首先，下载twain-dsm**

下载地址：[twain-dsm](https://github.com/twain/twain-dsm)

twain-dsm ： 此项目由C++开发，支持Windows/Linux/MacOS，实现了对Source Manager操作功能的封装。本人在Windows下用VS2017编译，进入twain-dsm/TWAIN_DSM/visual_studio/目录，打开对应的版本，把下面的代码注释掉，在属性中选择对应的Windows SDK版本，就可以编译了。

```
CTwnDsmLog::Log(){

​	// assert(0); // 注释掉此行，作用是当发生错误时弹出崩溃窗口

}
```

**（2）其次，下载对应的Application版本**

比如C#程序选择[twain-cs](https://github.com/twain/twain-cs)

C++程序选择[twain-samples](https://github.com/twain/twain-samples)

C程序选择[twain-specification](https://github.com/twain/twain-specification)

等等。

我选择的C#版本，用VS2017编译后，记得把之前编译好的TWAINDSM.dll文件复制到twain-cs/bin/debug目录下，与exe文件同一目录，之后就可以运行了。



**备注**：如果你使用的是C#版本，你要把代码集成到你自己的程序里面，记得TWAIN对象必须建立在Form对话框之下，因此TWAIN内部使用了对话框的事件循环，否则将会遇到程序卡死的情况。

