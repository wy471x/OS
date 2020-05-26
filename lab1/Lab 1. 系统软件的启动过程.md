## Lab 1. 系统软件的启动过程

 实验目的：
    操作系统是一个软件，也需要通过某种机制加载并运行它。在这里我们将通过另外一个更加 简单的软件-bootloader来完成这些工作。为此，我们需要完成一个能够切换到x86的保护模式 并显示字符的bootloader，为启动操作系统ucore做准备。lab1提供了一个非常小的bootloader 和ucore OS，整个bootloader执行代码小于512个字节，这样才能放到硬盘的主引导扇区中。通过分析和实现这个bootloader和ucore OS，读者可以了解到：

* 计算机原理

  * CPU的编址与寻址: 基于分段机制的内存管理 
  * CPU的中断机制 
  * 外设：串口/并口/CGA，时钟，硬盘 

* Bootloader软件 

  * 编译运行bootloader的过程 
  * 调试bootloader的方法 
  * PC启动bootloader的过程 
  * ELF执行文件的格式和加载 
  * 外设访问：读硬盘，在CGA上显示字符串

* ucore OS软件

  * 编译运行ucore OS的过程 
  * ucore OS的启动过程 
  * 调试ucore OS的方法 
  * 函数调用关系：在汇编级了解函数调用栈的结构和处理过程 
  * 中断管理：与软件相关的中断处理 
  * 外设管理：时钟 

实验内容： 
    lab1中包含一个bootloader和一个OS。这个bootloader可以切换到X86保护模式，能够读磁盘 并加载ELF执行文件格式，并显示字符。而这lab1中的OS只是一个可以处理时钟中断和显示 字符的幼儿园级别OS。

---

### 练习：

为了实现lab1的目标，lab1提供了6个基本练习和1个扩展练习，要求完成实验报告。 

对实验报告的要求： 

* 基于markdown格式来完成，以文本方式为主。 
* 填写各个基本练习中要求完成的报告内容 
* 完成实验后，请分析ucore_lab中提供的参考答案，并请在实验报告中说明你的实现与参考答案的区别 
* 列出你认为本实验中重要的知识点，以及与对应的OS原理中的知识点，并简要说明你对 二者的含义，关系，差异等方面的理解（也可能出现实验中的知识点没有对应的原理知识点） 
* 列出你认为OS原理中很重要，但在实验中没有对应上的知识点

#### 练习1. 理解通过make生成执行文件的过程。（要求在报告中写出对下述问题的回答）

列出本实验各练习中对应的OS原理的知识点，并说明本实验中的实现部分如何对应和体现了 

原理中的基本概念和关键知识点。 

在此练习中，大家需要通过静态分析代码来了解： 

1. **操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)**

   ![1](https://github.com/wy471x/OS/blob/master/lab1/1.png)

   上图，可以看出Makefile文件执行时利用`gcc`将一些C的源文件（即带有`.c`后缀的文件）编译成目标文件（即带有`.o`后缀的文件）。

   ![2](https://github.com/wy471x/OS/blob/master/lab1/2.png)

   上图，可以看出Makefile文件执行时利用`ld`将目标文件转换成执行程序（即带有`.out`后缀的文件）。

   ![3](https://github.com/wy471x/OS/blob/master/lab1/3.png)

   上图，可以看出Makefile文件执行时利用`dd	`将可执行程序（即`bootloader`）放入一个虚拟硬盘中（即`ucore.img`）。


2. **一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？**

   ![4](https://github.com/wy471x/OS/blob/master/lab1/4.png)

   由上图可知，在该C源文件（`sign.c`）中描述了符合规范的硬盘主引导扇区的大小是512字节，并且末尾是以`0x55`和`0xAA`结束的。

补充材料： 

如何调试Makefile 

当执行make时，一般只会显示输出，不会显示make到底执行了哪些命令。 

如想了解make执行了哪些命令，可以执行：

``` shell
$ make "V=" 
```

要获取更多有关make的信息，可上网查询，并请执行 

```shell
$ man make
```

#### 练习2：使用qemu执行并调试lab1中的软件。（要求在报告中简要写出练习过程） 

为了熟悉使用qemu和gdb进行的调试工作，我们进行如下的小练习： 

1. **从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。** 

   在`lab1_result`目录下执行`make lab1-mon`后，会执行进入GDB调试界面，使用`ni`或`si`指令来执行单步跟踪BIOS的执行。

2. **在初始化位置0x7c00设置实地址断点,测试断点正常。**

   在`labcodes/lab1/tools/`下新建初始化文件`lab1init`，并在其中添加以下内容：
   ```shell
   file bin/kernel
   target remote :1234
   set architecture i8086
   b *0x7c00
   continue
   x/2i $pc
   ```
   ，内容添加完成后，切换至`labcodes/lab1`执行`make lab1-mon`即可。
3. **从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。** 
   在第二题的基础上，在GDB命令行下执行`si`或`ni`命令，经过观察发现每一步的执行命令与`bootasm.S`和`bootblock.asm`文件中的保持一致。
4. **自己找一个bootloader或内核中的代码位置，设置断点并进行测试。**

   在`labcodes/lab1/tools/`下的`lab1init`文件修改断点位置为`0x7c0e`，执行`make lab1-mon`验证`0x7c0e`断点的位置设置正确并正常执行。

> 提示：参考附录“启动后第一条执行的指令”，可了解更详细的解释，以及如何单步调试和 
>
> 查看BIOS代码。 
>
> 提示：查看 labcodes_answer/lab1_result/tools/lab1init 文件，用如下命令试试如何调试 
>
> bootloader第一条指令： 
>
> ```shell
> $ cd labcodes_answer/lab1_result/ 
> $ make lab1-mon
> ```

补充材料： 我们主要通过硬件模拟器qemu来进行各种实验。在实验的过程中我们可能会遇上 

各种各样的问题，调试是必要的。qemu支持使用gdb进行的强大而方便的调试。所以用好 

qemu和gdb是完成各种实验的基本要素。 

默认的gdb需要进行一些额外的配置才进行qemu的调试任务。qemu和gdb之间使用网络端口 

1234进行通讯。在打开qemu进行模拟之后，执行gdb并输入 

```shell
target remote localhost:1234
```

即可连接qemu，此时qemu会进入停止状态，听从gdb的命令。 

另外，我们可能需要qemu在一开始便进入等待模式，则我们不再使用make qemu开始系统的 

运行，而使用make debug来完成这项工作。这样qemu便不会在gdb尚未连接的时候擅自运行 

了。

gdb的地址断点 

在gdb命令行中，使用b *[地址]便可以在指定内存地址设置断点，当qemu中的cpu执行到指定 

地址时，便会将控制权交给gdb。 

关于代码的反汇编 

有可能gdb无法正确获取当前qemu执行的汇编指令，通过如下配置可以在每次gdb命令行前强 

制反汇编当前的指令，在gdb命令行或配置文件中添加： 

练习

```shell
define hook-stop 
x/i $pc 
end
```

即可

gdb的单步命令 

在gdb中，有next, nexti, step, stepi等指令来单步调试程序，他们功能各不相同，区别在于单 

步的“跨度”上。 

```shell
next 单步到程序源代码的下一行，不进入函数。 
nexti 单步一条机器指令，不进入函数。 
step 单步到下一个不同的源代码行（包括进入函数）。 
stepi 单步一条机器指令。
```

#### 练习3：分析bootloader进入保护模式的过程。（要求在报告中写出分析）

BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。
提示：需要阅读小节“保护模式和分段机制”和lab1/boot/bootasm.S源码，了解如何从实模式切换到保护模式，需要了解：

* **为何开启A20，以及如何开启A20**

  参见附录“关于A20 Gate”

* **如何初始化GDT表**

  参考`bootasm.S`源码

* **如何使能和进入保护模式**

  参考`bootasm.S`源码

#### 练习4：分析bootloader加载ELF格式的OS的过程。（要求在报告中写出分析）

通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运 行并调试bootloader&OS，

* bootloader如何读取硬盘扇区的？ 

* bootloader是如何加载ELF格式的OS？

提示：可阅读“硬盘访问概述”，“ELF执行文件格式概述”这两小节。

#### 练习5：实现函数调用堆栈跟踪函数	（需要编程）

我们需要在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数 print_stackframe来跟踪函数调用堆栈中记录的返回地址。在如果能够正确实现此函数，可在 lab1中执行	“make	qemu”后，在qemu模拟器中得到类似如下的输出：

```assembly
…… ebp:0x00007b28	eip:0x00100992	args:0x00010094	0x00010094	0x00007b58	0x00100096				kern/debug/kdebug.c:305:	print_stackframe+22 ebp:0x00007b38	eip:0x00100c79	args:0x00000000	0x00000000	0x00000000	0x00007ba8				kern/debug/kmonitor.c:125:	mon_backtrace+10 ebp:0x00007b58	eip:0x00100096	args:0x00000000	0x00007b80	0xffff0000	0x00007b84				kern/init/init.c:48:	grade_backtrace2+33 ebp:0x00007b78	eip:0x001000bf	args:0x00000000	0xffff0000	0x00007ba4	0x00000029				kern/init/init.c:53:	grade_backtrace1+38 ebp:0x00007b98	eip:0x001000dd	args:0x00000000	0x00100000	0xffff0000	0x0000001d				kern/init/init.c:58:	grade_backtrace0+23 ebp:0x00007bb8	eip:0x00100102	args:0x0010353c	0x00103520	0x00001308	0x00000000				kern/init/init.c:63:	grade_backtrace+34 ebp:0x00007be8	eip:0x00100059	args:0x00000000	0x00000000	0x00000000	0x00007c53				kern/init/init.c:28:	kern_init+88 ebp:0x00007bf8	eip:0x00007d73	args:0xc031fcfa	0xc08ed88e	0x64e4d08e	0xfa7502a8 
<unknow>:	--	0x00007d72	—
……
```

请完成实验，看看输出是否与上述显示大致一致，并解释最后一行各个数值的含义。

提示：可阅读小节“函数堆栈”，了解编译器如何建立函数调用关系的。在完成lab1编译后，查 看lab1/obj/bootblock.asm，了解bootloader源码与机器码的语句和地址等的对应关系；查看 lab1/obj/kernel.asm，了解	ucore	OS源码与机器码的语句和地址等的对应关系。
要求完成函数kern/debug/kdebug.c::print_stackframe的实现，提交改进后源代码包（可以编 译执行），并在实验报告中简要说明实现过程，并写出对上述问题的回答。
补充材料：
由于显示完整的栈结构需要解析内核文件中的调试符号，较为复杂和繁琐。代码中有一些辅 助函数可以使用。例如可以通过调用print_debuginfo函数完成查找对应函数名并打印至屏幕的 功能。具体可以参见kdebug.c代码中的注释。

#### 练习6：完善中断初始化和处理	（需要编程）

请完成编码工作和回答如下问题：

1. 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪 几位代表中断处理代码的入口？ 

2. 请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中， 依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个 中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。

3. 请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中 处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向 屏幕上打印一行文字”100	ticks”。

> 【注意】除了系统调用中断(T_SYSCALL)使用陷阱门描述符且权限为用户态权限以外， 其它中断均使用特权级(DPL)为０的中断门描述符，权限为内核态权限；而ucore的应用 程序处于特权级３，需要采用｀int	0x80`指令操作（这种方式称为软中断，软件中断， Tra中断，在lab5会碰到）来发出系统调用请求，并要能实现从特权级３到特权级０的转 换，所以系统调用中断(T_SYSCALL)所对应的中断门描述符中的特权级（DPL）需要设 置为３。

要求完成问题2和问题3	提出的相关函数实现，提交改进后的源代码包（可以编译执行），并 在实验报告中简要说明实现过程，并写出对问题1的回答。完成这问题2和3要求的部分代码 后，运行整个系统，可以看到大约每1秒会输出一次”100	ticks”，而按下的键也会在屏幕上显示。

提示：可阅读小节“中断与异常”。
