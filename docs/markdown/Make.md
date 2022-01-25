# Make说明

> 在linux系统中make是一个非常重要的编译命令，不管是自己进行项目开发还是安装应用软件，我们都经常要用到make或makeinstall。利用make工具，我们可以将大型的开发项目分解成为多个更易于管理的模块，一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，Makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。makefile 带来的好处就是“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。

## Make的工作原理

当make 命令被执行时，它会扫描当前目录下Makefile或makefile文件找到目标以及其依赖。如果这些依赖自身也是目标，继续为这些依赖扫描Makefile 建立其依赖关系，然后编译它们。一旦主依赖编译之后，然后就编译主目标,假设你对某个源文件进行了修改，你再次执行make 命令，它将只编译与该源文件相关的目标文件，因此，编译完最终的可执行文件节省了大量的时间。

# Make命令

|  参数  |                            解释                            |
| :----: | :--------------------------------------------------------: |
|   -f   |                     指定“makefile”文件                     |
|   -i   |                 忽略命令执行返回的出错信息                 |
|   -s   |         沉默模式，在执行之前不输出相应的命令行信息         |
|   -r   |                    禁止使用build-in规则                    |
|   -n   |          非执行模式，输出所有执行命令，但并不执行          |
|   -t   |                        更新目标文件                        |
|   -q   | make操作将根据目标文件是否已经更新返回"0"或非"0"的状态信息 |
|   -p   |                输出所有宏定义和目标文件描述                |
|   -d   |        Debug模式，输出有关文件和检测时间的详细信息         |
| -C dir |           在读取makefile 之前改变到指定的目录dir           |
| -l dir |      当包含其他makefile文件时，利用该选项指定搜索目录      |
|   -h   |                help文挡，显示所有的make选项                |
|   -w   |          在处理makefile之前和之后，都显示工作目录          |

# Makefile的写法

> 学习make工具，需要明白三个概念：目标、依赖、处理动作。
> makefile所要进行的主要内容是明确目标、明确目标所依赖的内容、明确依赖条件满足时应该执行对应的处理动作。例如我们最终要实现a这个目标，但是需要依赖b，而b依赖于c的存在，则可以描述为：
>
> ```makefile
> a:b
> 	cmdbtoa
> b:c
> 	cmdctob
> ```
>
> “：”代表依赖，另外每个处理动作之前都要用tab键分隔。
> 上述四行的意思是：a依赖于b，而处理cmdbtoa；b依赖于c，而处理cmdctob。

## 实例

首先准备三个文件，test1.c、test2.c、test2.h。

test1.c

```c
#include<stdio.h>
#include "test2.h"

int main() 
{
    printf("This is test1!\n");
    PrintTest2();
    return 0;
}
```


test2.c

```c
#include<stdio.h>
#include "test2.h"
 
void PrintTest2() 
{
    printf("This is test2!\n");
}
```
test2.h 

```c
#ifndef TEST2_H_
#define TEST2_H_

void PrintTest2();

#endif
```

使用ls指令查看目录内容：

有了上述三个代码文件后，下面介绍使用三种makefie文件编写方式

### **1、基础版**

新建一个文件，命名为Makefile，文件内容如下（注意要用tab来分隔处理动作指令）：

```makefile
test: test1.o test2.o 
	gcc -Wall test1.o test2.o -o test
    
test1.o: test1.c test2.h
	gcc -c -Wall test1.c -o test1.o

test2.o: test2.c test2.h
	gcc -c -Wall test2.c -o test2.o

clean: 
    rm -rf *.o test
```

然后在终端内输入make指令编译工程：


再次使用ls查看目录多了.o和可执行文件：

对makefile里的指令进行解释：

> test: test1.o test2.o ：test依赖于test1.o、test2.o两个目标文件
>
> gcc -Wall test1.o test2.o -o test：编译出可执行文件test。-o表示指定的可执行文件名称
>
> test1.o: test1.c test2.h：test1.o依赖于test1.c、test2.h两个文件
>
> gcc -c -Wall test1.c -o test1.o：编译出test1.o文件，-c表示只把给它的文件编译成目标文件，用源码文件的文件名命名，但把其后缀由“.c”变成“.o”。
>
> make clean：删除掉所有.o以及可执行文件，当修改了某一个源文件使用该指令清除旧的文件。在终端输入“make clean”实现：


此时修改了test2.c中内容，在printf（）下一行增加一个无意义的回车，然后使用make编译会发现只有跟test2.c有关的部分进行了处理，这种自动化的进行依赖性检测和选择性执行必要的处理动作的工作，体现了make的特色

### **2、使用变量版**

makefile内容如下：

```makefile
OBJS = test1.o test2.o
G = gcc
CFLAGS = -Wall -O -g

test:$(OBJS)
	$(G) $(OBJS) -o test

test1.o:test1.c test2.h
	$(G) $(CFLAGS) -c test1.c
test2.o:test2.c test2.h
	$(G) $(CFLAGS) -c test2.c

clean:
	rm -rf *.o test
```

在这个makefile中使用了变量，变量的格式是**“varName = content”**；若要引用这个变量，只需要用**$** 即可，**$(varName)**。

> CFLAGS = -Wall -O -g 配置编译器设置，并把它赋值给CFLAGS变量
>
> -Wall：输出所有警告信息
>
> -O：在编译时进行优化
>
> -g：表示编译debug版本

使用make后生成文件同上

这样写的Makefile比较简单，但是缺点显著，每添加一个.c文件，就需要修改Makefile文件。

### **3、使用函数**

makefile内容如下

```makefile
C = gcc
G = g++
CFLAGS = -Wall -O -g
TARGET = ./test

%.o:%.c
	$(C) $(CFLAGS) -c $< -o $@

%.o:%.cpp
	$(G) $(CFLAGS) -c $< -o $@

SOURCES = $(wildcard *.c *.cpp)

OBJS = $(patsubst %.c,%.o,$(patsubst %.cpp,%.o,$(SOURCES)))

$(TARGET):$(OBJS)
	$(G) $(OBJS) -o $(TARGET)
	chmod a+x $(TARGET)

clean:
	rm -rf *.o test
```

>
> %.o:%.c
> $© $(CFLAGS) -c $< -o $@
>
> %.o:%.cpp
> $(G) $(CFLAGS) -c $< -o $@
>
> 表示把所有的.c、.cpp文件编译成.o文件。
>
> SOURCES = $(wildcard *.c *.cpp) 表示产生一个所有以.c、.cpp结尾的文件列表，然后存入变量SOURCES里。
>
> OBJS = $ (patsubst %.c,%.o,$ (patsubst %.cpp,%.o,$(SOURCES)))
> 把SOURCES文件列表中所有.cpp变成.o、.c变成.o，然后形成一个新的列表，存入OBJS变量。
>
> $ @扩展成当前规则的目的文件名，$ <扩展成依靠列表中的第一个依靠文件，$^扩展成整个依靠的列表（除掉里面所有重复的文件名）。
>
> chmod a+x $(TARGET) 表示把firstTest强制变成可执行文件。

使用make后生成文件同上