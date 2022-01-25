## 1. g++必备基础

> 在学习CMake和和Makefile之前我们先学下g++这个工具，大家或许会问为什么要学g++，不应该直接学CMake和Makefile吗。实际上如果你不掌握g++根本就不会写Makefile，因为它实际上就是对g++代码的整理，有了Makefile，执行程序会更加快速方便。另外CMake就是为了简化Makefile的编写，它可以自动生成Makefile。

### 1.1 安装g++

我们在安装g++之前可以看一下自己是否已经安装了g++，因为ubuntu安装后就默认安装了g++，下面命令可查看自己g++版本。

**Tips：如果不想作死，就不要手贱去降级或者升级g++版本。**

`g++ --version`

![img](Photo/15961.jpg)

因为我已经安装了g++，出现了上面安装的版本号。如果你出现了上面信息，就不需要再安装了，没有的话，用下面的命令即可完成安装。

`sudo apt-get install g++`

安装好后也可以通过`g++ --version`查看是否安装成功

### **1.2 编译流程**

现在我们已经安装好了g++，接下来通过写一个简单的程序来看看整个的编译流程。

我们通过vim创建一个test.cpp文件，测试的代码如下：

```c++
#include <iostream>
using namespace std;

int main()
{
	cout << "Hello, world!" << endl;
	return 0;
}
```

![img](Photo/65083.jpg)

测试代码完成后，我们来进行下编译，打开终端，在终端输入g++ 文件名即可，在这个程序中就是下面命令：

`g++ test.cpp`

> 注意这里的文件名是包括路径的，要是不知道文件路径的话可以在敲完g++和空格之后直接把文件拖进去，系统会自动添加文件路径。

在终端完成上面的命令后，你发现并没有任何输出，但这时候你去主文件夹下（默认主文件夹）看下会发现有个a.out文件

![img](Photo/93033.jpg)

现在你再在终端输入下面命令就能看到结果。

`./a.out`

![img](Photo/63995.jpg)

接下来我来解释下这个.out文件，实际上这是个经过相应的链接产生的可执行文件。还有个.o文件，它是个中间文件，一般是通过编译的但还未链接。我们通过看看g++在执行编译工作的时候的流程，你就会有更好的理解。如下：

1. 预处理，生成.i的文件
2. 将预处理后的文件转换成汇编语言，生成.s文件
3. 将汇编变为目标代码(机器代码)，生成.o的文件
4. 连接目标代码,生成可执行程序

对于这个流程，我们结合上面的例子，再详细介绍下，如下：

#### 1.预处理阶段

首先在终端输入下面代码：

`g++ -E test.cpp > test.i`

预处理后的文件在 linux下以.i为后缀名，这个过程是用来激活预处理，执行完命令后，你会发现主文件夹下多了一个test.i文件

![img](Photo/62729.jpg)

这一步（预处理）主要做了宏的替换，和注释的消除。

![img](Photo/34183.jpg)

上图是test.i文件的最后部分，可以看见宏的替换和注释的消除。

#### 2.将预处理后的文件转换成汇编语言

在终端输入下面代码：

`g++ -S test.cpp`

这一步主要就是生成test.s文件，.s文件表示汇编文件，用编辑器打开就都是汇编指令。下图是test.s文件的一部分。

![img](Photo/78931.jpg)

#### 3.将汇编语言变为目标代码(机器代码)

在终端输入下面代码：

`g++ -c test.cpp`

这一步就是生成目标文件，用编辑器打开就都是二进制机器码。

![img](Photo/32108.jpg)

#### 4.链接目标代码，生成可执行程序

在终端输入下面代码：

`g++ test.o -o test`

在这一步中生成的可执行程序名为test，如果执行命令 g++ test.o 这样默认生成a.out

![img](Photo/29326.jpg)

最后我们再看下这个过程中产生的所有文件，如下：

![img](Photo/20424.jpg)

### 1.3 总结

> gcc编译源文件到可执行文件的顺序

1. 通过`g++ -E 源文件`将文件预处理输出到控制台，`g++ -E 源文件 > 源文件.i`后加 `> 源文件.i`将文件输出到 源文件.i （预处理后的文件在 linux下以`.i`为后缀名）
2. 通过`g++ -S 源文件`输出以`.s`结尾的汇编文件
3. 通过`g++ -c 源文件`生成以`.o`为结尾的二进制目标文件
4. 通过`g++ 源文件.o`默认生成名为a（Linux下为.out，Windows下为.exe）的可执行文件可通过`-o`修改输出文件名

这就是编译的整个过程，你掌握了吗，这个过程对于后面编写Makefile非常重要，一定要深刻理解。

## 2. Makefile必备基础

[Make](./Make.md)

上面我们对g++和编译过程进行了介绍，现在我们继续学习如何编写Makefile。

### 2.1 Makefile介绍

Makefile描述了整个工程的编译、链接等规则，它定义了一系列规则来指定哪些文件需要编译以及如何编译、需要创建哪些库文件以及如何创建这些库文件、如何产生我们想要的可执行文件。

而且Makefile可以有效的减少大工程中需要编译和链接的文件，只编译和链接那些需要修改的文件，可以说使用Makefile，整个工程都可以完全自动化编译。

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fr8ya1mzfmc.jpeg%253FimageView2%252F2%252Fw%252F1620)

### 2.2 Makefile基本格式

target ... : prerequisites ... command ... ...

target - 目标文件, 可以是 Object File, 也可以是可执行文件

prerequisites - 生成target所需要的文件或者目标

command - make需要执行的命令(任意的shell命令)，Makefile中的命令必须以 [tab] 开头

### 2.3 Makefile语法

Makefile包含了五个重要的东西：显示规则、隐晦规则、变量定义、文件指示和注释。详细解释如下：

1. 显示规则：
   通常在写makefile时使用的都是显式规则，这需要指明target和prerequisite文件。一条规则可以包含多个target，这意味着其中每个target的prerequisite都是相同的。当其中的一个target被修改后，整个规则中的其他target文件都会被重新编译或执行。
2. 隐晦规则：
   make的自动推导功能所执行的规则
3. 变量的定义：
   Makefile中定义的变量，一般是字符串
4. 文件指示：
   Makefile中引用其他Makefile；指定Makefile中有效部分；定义一个多行命令
5. 注释：
   Makefile只有行注释 "#", 如果要使用或者输出"#"字符, 需要进行转义, "\#

### 2.4 Makefile简单实例

尽管上面介绍了许多Makefile的知识点，但我相信一定你很晕，接下来我通过一个实例来说明如何编写Makefile。

#### 2.4.1 准备程序文件

我们使用opencv对下面这只可爱的猫进行读取显示。

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fzbsiqb5362.jpeg%253FimageView2%252F2%252Fw%252F1620)

在这里我们用c++和opencv对图片进行读取和显示，程序保存在DisplayImage.cpp这个文件里，代码如下：

```c++
#include <stdio.h>
#include <opencv2/opencv.hpp>
using namespace cv;
int main(int argc, char** argv)
{
    if (argc != 2)
    {
        printf("usage: DisplayImage.out <Image_Path>\n");
        return -1;
    }
    Mat image;
    image = imread(argv[1], 1);
    if (!image.data)
    {
        printf("No image data \n");
        return -1;
    }
    namedWindow("Display Image", WINDOW_AUTOSIZE);
    imshow("Display Image", image);
    waitKey(0);
    return 0;
}
```

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fdra78zeo4e.jpeg%253FimageView2%252F2%252Fw%252F1620)

#### 2.4.2 Makefile编写

上面我们已经准备好了.cpp文件，现在我们来编写Makefile进而进行编译，程序如下：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fzkjel2s2u7.jpeg%253FimageView2%252F2%252Fw%252F1620)

现在我来解释下应该如何编写这个Makefile，对于编写Makefile我建议从下往上写。步骤如下：

##### 1.编写clean

这一步在Makefile中基本差不多，它的作用就是删除所有的.o文件和可执行文件。为什么这样做呢?我举个例子说明下，如果你有100个.cpp文件，经过编译后会得到一个可执行文件。在这个过程中我们会得到许多不必要的文件，例如100个.o文件，但这个文件又没有用，如果用rm的话那就太麻烦了，所以我们用了clean，它可以很轻松完成这个任务。另外请注意Makefile文件在执行时不会执行clean这个命令，需要我们调用才会执行，即make clean。clean代码如下：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Ftoxce4bmir.png%253FimageView2%252F2%252Fw%252F1620)

##### 2.编写目标文件1：依赖文件1

目标文件就是你想得到的文件，依赖文件就是你目前所拥有的东西。在本实例中我们现在拥有DisplayImage.cpp，所以DisplayImage.cpp是依赖文件，我们想得到DisplayImage.o，所以它是目标文件。代码如下：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252F9uhaukclr.png%253FimageView2%252F2%252Fw%252F1620)

##### 3.编写目标文件2：依赖文件2

这一步的依赖文件2实际就是第二步的目标文件1，在第二步我们通过DisplayImage.cpp得到了DisplayImage.o，现在我们需要通过DisplayImage.o得到可执行文件DisplayImage。所以在这一步目标文件是DisplayImage，依赖文件是DisplayImage.o，代码如下：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fux2p3w4mp6.png%253FimageView2%252F2%252Fw%252F1620)

##### 4.应用opencv库和头文件

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fpdj8m2nudk.png%253FimageView2%252F2%252Fw%252F1620)

这一步就需要根据自己计算机来配置了，对于我们初学者来说挺麻烦的，可以自己尝试下。有问题可以联系我们。

编写完makefile后，我们在终端make下就行了。下面编译后的文件：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Faf5lxq3nsg.png%253FimageView2%252F2%252Fw%252F1620)

最后在终端输入下面代码即可显示图片。

./DisplayImage 01.jpg

总体来说编写Makefile可以按照这个套路写，多写几次就会了。

## 3. CMake必备基础

说完Makefile，我们再说下CMake。CMake是一个跨平台的编译(Build)工具，可以用简单的语句来描述所有平台的编译过程，其是在make基础上发展而来的，早期的make需要程序员写Makefile文件，进行编译，而现在CMake能够通过对cmakelists.txt的编辑，轻松实现对复杂工程的组织。下面我带大家学习下CMake的基础知识。

### 3.1 安装CMake

首先我们看看如何在自己的linux系统(我的系统Ubuntu18.04)下安装CMake。方法如下：

`sudo apt-get install cmake`

输入上面命令后实际上就安装成功了，可以通过下面命令来检查：

`cmake --version`

如果你的界面如下图所示即说明安装成功。

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fcgr856d16g.png%253FimageView2%252F2%252Fw%252F1620)

### 3.2 CMake编译流程

成功安装好CMake后我们再来说说如何在linux平台下使用CMake生成Makefile并编译的流程，如下：

1. 编写CMake配置文件CMakeLists.txt，我们可以认为CMakeLists.txt就是CMake所处理的"代码"。
2. 执行命令 cmake path生成Makefile,其中path是CMakeLists.txt所在的目录。
3. 使用make命令进行编译。

### 3.3 使用CMake编译程序

我们通过一个关于opencv读取图片的程序，让大家更好的理解整个CMake的编译过程。

#### 3.3.1 准备程序文件

这里程序准备可以按照第二部分makefile那里准备。最后文件目录结构如下：

├── build

├── CMakeLists.txt

├── DisplayImage.cpp

opencv读取图片的程序写完后，我们需要编写CMake处理的代码了，即CMakeLists.txt。

#### 3.3.2 编写CMakeLists.txt

现在我们编写CMakeLists.txt文件，该文件实际上放在哪里都可以，只要编写的路径能够正确指向就好了，CMakeLists.txt文件内容如下所示：

```cmake
cmake_minimum_required(VERSION 2.8)

project( DisplayImage )

find_package( OpenCV REQUIRED )

add_executable( DisplayImage DisplayImage.cpp )

target_link_libraries( DisplayImage ${OpenCV_LIBS} )
```

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fsjhvpyl6vc.png%253FimageView2%252F2%252Fw%252F1620)

看到这些代码是不是很闷逼，为了让大家明白CMakeLists.txt文件内容，接下来我说一下Cmake的一些常用命令，你就能很好的理解上面的代码了。

1）`cmake_minimum_required` 命令

命令语法：`cmake_minimum_required(VERSION major[.minor[.patch[.tweak]]][FATAL_ERROR])`

命令简述：用于指定需要的CMake 的最低版本

使用范例：`cmake_minimum_required(VERSION 2.8)`

2）`project` 命令

命令语法：`project(<projectname> [languageName1 languageName2 … ] )`

命令简述：用于指定项目的名称，一般和项目的文件夹名称对应

使用范例：p`roject(DisplayImage)`

3）`aux_source_directory` 命令

命令语法：`aux_source_directory(<dir> <variable>)`

命令简述：用于将 dir 目录下的所有源文件的名字保存在变量 variable 中

使用范例：`aux_source_directory(src DIR_SRCS)`

4）`add_executable` 命令

命令语法：`add_executable(<name> [WIN32] [MACOSX_BUNDLE][EXCLUDE_FROM_ALL] source1 source2 … sourceN)`

命令简述：用于指定从一组源文件 source1 source2 … sourceN 编译出一个可执行文件且命名为name

使用范例:`add_executable( DisplayImage DisplayImage.cpp )`

5）`target_link_libraries` 命令

命令语法：`target_link_libraries(<target> [item1 [item2 […]]][[debug|optimized|general] ] …)`

命令简述：用于指定 target 需要的链接 item1 item2 …。这里 target 必须已经被创建，链接的 item 可以是已经存在的 target（依赖关系会自动添加）

使用范例：`target_link_libraries( DisplayImage ${OpenCV_LIBS} )`

6）`add_subdirectory` 命令

命令语法：`add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])`

命令简述：用于添加一个需要进行构建的子目录

使用范例：`add_subdirectory(Lib)`

7）`include_directories` 命令

命令语法：`include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 …)`

命令简述：用于设定目录，这些设定的目录将被编译器用来查找 include 文件

使用范例：`include_directories(${PROJECT_SOURCE_DIR}/lib)`

像这样的命令还有很多，如`find_package（）`寻找使用第三方库等，这些都需要我们平时多加积累。给大家一个查询命令的方法，大家可以多去看cmake官网的help，链接如下：

https://cmake.org/cmake/help/v2.8.8/cmake.html#section_Commands

### 3.3 编译和运行程序

现在CMakeLists.txt文件已经编写好了，意味着我们的工作即将进入尾声。现在看看我们的文件结构目录，如下图：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Flnx8l7c3rg.png%253FimageView2%252F2%252Fw%252F1620)

接下来我们就需要进行编译了。编译的过程相对于CMakeLists.txt文件的编写是很简单的，只有两步，如下

`cmake`

`make`

其中`cmake`命令将CMakeLists.txt文件转化为make所需要的makefile文件，最后用`make`命令编译源码生成可执行程序或共享库。对于我们这个实例，编译如下：

首先我们在命令行输入`cmake .`(注意cmake和.之间有空格)，表明Cmakelist.txt文件在当前目录下。

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fldlhxokdr8.jpeg%253FimageView2%252F2%252Fw%252F1620)

接下来在命令行输入`make`

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fm0g5yx81dm.jpeg%253FimageView2%252F2%252Fw%252F1620)

这样我们就编译成功了，我们看下编译后的文件目录

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fmjah4d2ro0.png%253FimageView2%252F2%252Fw%252F1620)

解释下这个build文件夹，由于cmake后会生成很多编译的中间文件以及makefile文件，所以一般建议新建一个新的目录，专门用来编译，这就是这里的build，打开build后，里面的文件如下：

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fnjsw80g4v3.png%253FimageView2%252F2%252Fw%252F1620)

到这里，我们不禁要问怎么没有图片显示呢，别急，在build目录下的命令行输入下面命令即可显示图片，这就是生产的DisplayImage可执行文件。

./DisplayImage ../01.jpg

![img](Photo/filtersno_upscale()imageUrl=https%253A%252F%252Fcubox.pro%252Fc%252Ffilters%253Ano_upscale()%253FimageUrl%253Dhttps%253A%252F%252Fask.qcloudimg.com%252Fhttp-save%252Fyehe-1508658%252Fldkfuqy0y9.png%253FimageView2%252F2%252Fw%252F1620)

到这里，关于CMake的一些基本操作就介绍的差不多了，其实对于CMake的学习我认为必须在实例中多加应用，才能更好的掌握，因为它的复杂命令太多了。

### 3.4 总结

CMake和Makefile的基础我们就介绍完了，对于这两个工具其实不是一时就能学会的，需要大量的实践积累才能游刃有余。