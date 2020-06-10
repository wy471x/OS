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
```assembly
#include <asm.h>

# Start the CPU: switch to 32-bit protected mode, jump into C.
# The BIOS loads this code from the first sector of the hard disk into
# memory at physical address 0x7c00 and starts executing in real mode
# with %cs=0 %ip=7c00.

.set PROT_MODE_CSEG,        0x8                     # kernel code segment selector
.set PROT_MODE_DSEG,        0x10                    # kernel data segment selector
.set CR0_PE_ON,             0x1                     # protected mode enable flag

# start address should be 0:7c00, in real mode, the beginning address of the running bootloader
.globl start
start:
.code16                                             # Assemble for 16-bit mode
    cli                                             # Disable interrupts
    cld                                             # String operations increment

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment

    # Enable A20:
    #  For backwards compatibility with the earliest PCs, physical
    #  address line 20 is tied low, so that addresses higher than
    #  1MB wrap around to zero by default. This code undoes this.
seta20.1:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.1
    #          -------  below made by dunk  ------
    # inb $0x64, %al: 将端口0x64状态读入到寄存器al中
    # testb $0x2, %al: 测试0x64端口的第二位是否为0
    # jnz seta20.1: 如果不为0，则跳回至seta20.1标签处继续执行
    # 三个操作的目的：测试8042的输入缓冲区是否为空
    movb $0xd1, %al                                 # 0xd1 -> port 0x64
    outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port
    #           -------  below made by dunk  ------
    # movb $0xd1, %al: 将0xd1写入寄存器al中
    # outb %al, $0x64: 向0x64端口发送0xd1命令
    # 两个操作的目的：发送写数据命令(即：0xd1)到输入缓冲中

seta20.2:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.2

    movb $0xdf, %al                                 # 0xdf -> port 0x60
    outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
    #      ------ below made by dunk ------
    # movb $0xdf, %al: 将0xdf命令写入寄存器al中
    # outb %al, $0x60: 向0x60端口写入0xdf
    # 两个操作的目的：将8042 Output Port(P2)得到的字节的第2位置1，然后写入8042 Input buffer

    # Switch from real to protected mode, using a bootstrap GDT
    # and segment translation that makes virtual addresses
    # identical to physical addresses, so that the
    # effective memory map does not change during the switch.
    lgdt gdtdesc
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0
    #       ------ below made by dunk ------
    # lgdt gdtdesc: 加载GDT全局描述符表
    # movl %cr0, %eax: 将控制寄存器cr0中值写入通用寄存器eax中
    # orl $CR0_PE_ON, %eax: 将CR0_PE_ON值与控制寄存器中的值进行或操作，从而将控制寄存器中
    #                       控制保护模式开启的位置为1，即使能控制寄存器中保护模式位
    # movl %eax, %cr0: 将通用寄存器中的值写回至控制寄存器cr0
    # 上三步操作的目的：使能控制寄存器cr0中的保护模式位

    # Jump to next instruction, but in 32-bit code segment.
    # Switches processor into 32-bit mode.
    ljmp $PROT_MODE_CSEG, $protcseg

.code32                                             # Assemble for 32-bit mode
protcseg:
    # Set up the protected-mode data segment registers
    movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
    movw %ax, %ds                                   # -> DS: Data Segment
    movw %ax, %es                                   # -> ES: Extra Segment
    movw %ax, %fs                                   # -> FS
    movw %ax, %gs                                   # -> GS
    movw %ax, %ss                                   # -> SS: Stack Segment
    #     ------ below made by dunk ------
    # ljmp $PROT_MODE_CSEG, $protcseg: 跳转至下一条指令 解释详见：https://stackoverflow.com/questions/5211541/bootloader-switching-processor-to-protected-mode

    # Set up the stack pointer and call into C. The stack region is from 0--start(0x7c00)
    movl $0x0, %ebp
    movl $start, %esp
    call bootmain
    #    ------ below made by dunk ------
    # movl $0x0, %ebp: 将0x0写入寄存器ebp
    # movl $start, %esp: 将start标签处地址值写入寄存器esp
    # call bootmain: 调用bootmain函数
    # esp 和 ebp的区别：esp是当前栈的指针，ebp则是当前栈帧的基指针，参考：https://stackoverflow.com/questions/15020621/what-is-between-esp-and-ebp

    # If bootmain returns (it shouldn't), loop.
spin:
    jmp spin

# Bootstrap GDT
.p2align 2                                          # force 4 byte alignment
gdt:
    SEG_NULLASM                                     # null seg
    SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)           # code seg for bootloader and kernel
    SEG_ASM(STA_W, 0x0, 0xffffffff)                 # data seg for bootloader and kernel

gdtdesc:
    .word 0x17                                      # sizeof(gdt) - 1
    .long gdt                                       # address gdt

```

* **为何开启A20，以及如何开启A20**

  为了保持和8086的兼容，PC机在设计上在第21条地址线上做了一个开关，当这个开关打开时，这条地址线和其它地址线一样可以使用，当这个开关关闭时，第21条地址线恒为0，这个开关就叫做A20 Gate。在实模式下，若要访问高端内存区，此开关必须打开；同样在保护模式下。由于使用32位地址线，如果A20恒为0，那么系统只能访问奇数兆的内存，故在此模式下A20开关也必须开启。在保护模式下，为了使能所有地址位的寻址能力，通过向键盘控制器8042发送一个命令来完成对A20的开启操作。

* **如何初始化GDT表**

  位于`lab1\bootasm.S`中，全局描述符表如下定义的：
  ```assembly
    SEG_NULLASM                                     # null seg
    SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)           # code seg for bootloader and kernel
    SEG_ASM(STA_W, 0x0, 0xffffffff)                 # data seg for bootloader and kernel
  ```
  位于`lab1\kern\pmm.c`中，全局描述符表如下定义：
  ```assembly
    static struct segdesc gdt[] = {
    SEG_NULL,
    [SEG_KTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_KERNEL),
    [SEG_KDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_KERNEL),
    [SEG_UTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_USER),
    [SEG_UDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_USER),
    [SEG_TSS]    = SEG_NULL,
    }; 
  ```
  
* **如何使能和进入保护模式**


#### 练习4：分析bootloader加载ELF格式的OS的过程。（要求在报告中写出分析）

通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运 行并调试bootloader&OS，
```assembly
#include <defs.h>
#include <x86.h>
#include <elf.h>

/* *********************************************************************
 * This a dirt simple boot loader, whose sole job is to boot
 * an ELF kernel image from the first IDE hard disk.
 *
 * DISK LAYOUT
 *  * This program(bootasm.S and bootmain.c) is the bootloader.
 *    It should be stored in the first sector of the disk.
 *
 *  * The 2nd sector onward holds the kernel image.
 *
 *  * The kernel image must be in ELF format.
 *
 * BOOT UP STEPS
 *  * when the CPU boots it loads the BIOS into memory and executes it
 *
 *  * the BIOS intializes devices, sets of the interrupt routines, and
 *    reads the first sector of the boot device(e.g., hard-drive)
 *    into memory and jumps to it.
 *
 *  * Assuming this boot loader is stored in the first sector of the
 *    hard-drive, this code takes over...
 *
 *  * control starts in bootasm.S -- which sets up protected mode,
 *    and a stack so C code then run, then calls bootmain()
 *
 *  * bootmain() in this file takes over, reads in the kernel and jumps to it.
 * */

#define SECTSIZE        512
#define ELFHDR          ((struct elfhdr *)0x10000)      // scratch space

/* waitdisk - wait for disk ready */
static void
waitdisk(void) {
    while ((inb(0x1F7) & 0xC0) != 0x40)
        /* do nothing */;
}

/* readsect - read a single sector at @secno into @dst */
/*      ------ below made by dunk ------
		bootloader 读取扇区的步骤：
		1. 从0x1F7端口读硬盘状态，直到硬盘非忙
		2. 向0x1F2端口发送读写的扇区数量
		3. 向0x1F3 ~ 0x1F6端口发送LBA参数
		4. 向0x1F7端口发送读扇区的指令
		5. 从0x1F7端口读硬盘状态，直到硬盘不忙
		6. 从0x1F0端口读入扇区内容至dst
*/
static void
readsect(void *dst, uint32_t secno) {
    // step 1:
	// wait for disk to be ready
    waitdisk();
	// step 2:
    outb(0x1F2, 1);                         // count = 1
	// step 3:
    outb(0x1F3, secno & 0xFF);
    outb(0x1F4, (secno >> 8) & 0xFF);
    outb(0x1F5, (secno >> 16) & 0xFF);
    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);
	// step 4:
    outb(0x1F7, 0x20);                      // cmd 0x20 - read sectors
	
	// step 5:
    // wait for disk to be ready
    waitdisk();
	// step 6:
    // read a sector
    insl(0x1F0, dst, SECTSIZE / 4);
}

/* *
 * readseg - read @count bytes at @offset from kernel into virtual address @va,
 * might copy more than asked.
 * */
static void
readseg(uintptr_t va, uint32_t count, uint32_t offset) {
    uintptr_t end_va = va + count;

    // round down to sector boundary
    va -= offset % SECTSIZE;

    // translate from bytes to sectors; kernel starts at sector 1
    uint32_t secno = (offset / SECTSIZE) + 1;

    // If this is too slow, we could read lots of sectors at a time.
    // We'd write more to memory than asked, but it doesn't matter --
    // we load in increasing order.
    for (; va < end_va; va += SECTSIZE, secno ++) {
        readsect((void *)va, secno);
    }
}

/* bootmain - the entry of bootloader */
void
bootmain(void) {
    // read the 1st page off disk
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

    // is this a valid ELF?
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
    }

    struct proghdr *ph, *eph;

    // load each program segment (ignores ph flags)
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
    }

    // call the entry point from the ELF header
    // note: does not return
    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

bad:
    outw(0x8A00, 0x8A00);
    outw(0x8A00, 0x8E00);

    /* do nothing */
    while (1);
}
```

* bootloader如何读取硬盘扇区的？ 

  参考https://oscourse-tsinghua.gitbook.io/ucore-analysis/labs/lab1/practice4

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
