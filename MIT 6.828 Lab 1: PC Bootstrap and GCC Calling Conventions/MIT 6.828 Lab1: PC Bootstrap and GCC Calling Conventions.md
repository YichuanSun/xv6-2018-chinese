# Lab 1: Booting a PC

## Part 1: PC Bootstrap

### Getting started with x86 assembly x86汇编入门

汇编语言有两种格式，一种是AT&T格式，一种是Intel格式，两种格式不同。

### Simulating the x86 模拟x86处理器

### The PC's Physical Address Space PC的物理内存空间

### The ROM BIOS

在lab目录下打开两个terminal，其中一个输入 make qemu-gdb ，等待执行到自动停下，然后在另一个terminal中输入 make gdb ，就打开了gdb汇编调试器。



## Part 2: The Boot Loader

> 这个boot loader最后转到的地址是0x10000左右，而kernel加载到的真正地址是0x100000左右，并且这样也确实加载到了真实的kernel地址，这是为什么？
>
> 答：此时已经转为了保护模式，寻址方式已经发生了变化，所以这个地址不能只看代码中指出的地址，还要一系列转化才能指向正确的物理地址。
>
> boot.asm第408行为什么地址前会出现一个星号？
>
> 答：某种汇编格式，没查。

boot loader了。这三段代码看了好几遍，花了好几天才明白，也怪我太过于注重细节了，想把每个部分都搞清楚。



接part 1，BIOS加电自检系统外设等方面正常之后，从引导盘中读取boot loader到内存的固定地址0x7c00，之后boot loader做两件事，分别写在一段汇编**boot.s**和一段C语言程序**main.c**中

1. 汇编代码部分将实模式转变为保护模式。完成后调用C程序。
2. C程序将内核从硬盘读入内存某地址(0x10000)，并转到内核在内存的入口地址继续执行，BOIS将控制权移交到内核。、



**下面从boot.s开始分析**



```assembly
#include <inc/mmu.h>

# Start the CPU: switch to 32-bit protected mode, jump into C.
# The BIOS loads this code from the first sector of the hard disk into
# memory at physical address 0x7c00 and starts executing in real mode
# with %cs=0 %ip=7c00.

.set PROT_MODE_CSEG, 0x8         # kernel code segment selector
.set PROT_MODE_DSEG, 0x10        # kernel data segment selector
.set CR0_PE_ON,      0x1         # protected mode enable flag
```

设置几个必要的常量参数。



```assembly
.globl start
start:
  .code16                     # Assemble for 16-bit mode
  cli                         # Disable interrupts
  cld                         # String operations increment
```

设置全局标号start，表示汇编程序的起始地址。cli是关中断指令，关闭BIOS系统的中断响应，因为下面的内核代码不能被中断。cld是设置字节批量传输方向的标志位，具体可查阅movsb，movsw等字节批量传输汇编语言命令的说明。



```assembly
  # Set up the important data segment registers (DS, ES, SS).
  xorw    %ax,%ax             # Segment number zero
  movw    %ax,%ds             # -> Data Segment
  movw    %ax,%es             # -> Extra Segment
  movw    %ax,%ss             # -> Stack Segment
```

利用异或清零ax寄存器，并将ds、es、ss寄存器赋初值为0.



```assembly
  # Enable A20:
  #   For backwards compatibility with the earliest PCs, physical
  #   address line 20 is tied low, so that addresses higher than
  #   1MB wrap around to zero by default.  This code undoes this.
seta20.1:
  inb     $0x64,%al               # Wait for not busy
  testb   $0x2,%al
  jnz     seta20.1

  movb    $0xd1,%al               # 0xd1 -> port 0x64
  outb    %al,$0x64

seta20.2:
  inb     $0x64,%al               # Wait for not busy
  testb   $0x2,%al
  jnz     seta20.2

  movb    $0xdf,%al               # 0xdf -> port 0x60
  outb    %al,$0x60
```

设置A20，感兴趣可以<font color="red">查阅《x86汇编语言：从实模式到保护模式》P193</font>，说的很详细。这和我们的系统设计关系不大，可忽略。其中的几个IO接口0x64,0x60的含义，可以查阅[bochs XT, AT and PS/2	 I/O port addresses](http://bochs.sourceforge.net/techspec/PORTS.LST).这个文献同样来源于课程官方参考文献，链接可能会被墙。不建议初学者查阅，我花了很大功夫搞懂了他在做什么，却发现和系统设计的关系实在不大。



```
  # Switch from real to protected mode, using a bootstrap GDT
  # and segment translation that makes virtual addresses 
  # identical to their physical addresses, so that the 
  # effective memory map does not change during the switch.
  lgdt    gdtdesc
  movl    %cr0, %eax
  orl     $CR0_PE_ON, %eax
  movl    %eax, %cr0
  
  # Jump to next instruction, but in 32-bit code segment.
  # Switches processor into 32-bit mode.
  ljmp    $PROT_MODE_CSEG, $protcseg
```

英文注释已经说的很清楚了，将实模式转为保护模式。实模式与保护模式是什么，可以自行搜索，资料有的是。一句话，实模式是16位系统所用，为了支持32位以及更高位数的系统与应用，必须转到保护模式。

lgdt命令引入全局描述符表，用于保护模式寻址。cr0标志位就是os和计组课上学的，实模式转保护模式硬件实现的标志位，将cr0置为1，就从实模式转为了保护模式（当然，还需要一系列后续处理）。

跳转指令按32位代码段格式跳转，将处理器也转变为保护模式。



```assembly
  .code32                     # Assemble for 32-bit mode
protcseg:
  # Set up the protected-mode data segment registers
  movw    $PROT_MODE_DSEG, %ax    # Our data segment selector
  movw    %ax, %ds                # -> DS: Data Segment
  movw    %ax, %es                # -> ES: Extra Segment
  movw    %ax, %fs                # -> FS
  movw    %ax, %gs                # -> GS
  movw    %ax, %ss                # -> SS: Stack Segment
  
  # Set up the stack pointer and call into C.
  movl    $start, %esp
  call bootmain

```

初始化一系列32位保护模式对应的寄存器值，start标号压栈，调用C函数bootmain，转到了C语言程序段的执行。



**下面是main.c的分析，比boot.s难。**



```c
#include <inc/x86.h>
#include <inc/elf.h>

#define SECTSIZE	512
#define ELFHDR		((struct Elf *) 0x10000) // scratch space

void readsect(void*, uint32_t);
void readseg(uint32_t, uint32_t, uint32_t);
```

略去了其中一大段注释文字。

先引入了两个关键头文件。elf头文件全称是executable linkable format，是linux执行文件的通用格式，<font color="red">详见《程序员的自我修养：链接、装载与库》P56</font>。

再定义了块大小：512字节。如计组中所学，磁盘中的文件是按块传输的。又定义了一个指向一个elf结构的指针，分配到了0x10000地址。<font color="red">关于elfhdr，请查阅elf.h头文件，或官方源代码line 0955.</font>这都可以从课程官网下载。

最后定义了两个函数，readsection和readsegment。按字面意思，第一个是读指定磁盘块到内存，第二个是读指定代码段到内存。



```c
void
bootmain(void)
{
	struct Proghdr *ph, *eph;		//定义了两个代码段头文件，用来标示起点和终点代码段

	// read 1st page off disk
	readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);	//从内核之后0

	// is this a valid ELF?
	if (ELFHDR->e_magic != ELF_MAGIC)
		goto bad;

	// load each program segment (ignores ph flags)
	ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);
	eph = ph + ELFHDR->e_phnum;
	for (; ph < eph; ph++)
		// p_pa is the load address of this segment (as well
		// as the physical address)
		readseg(ph->p_pa, ph->p_memsz, ph->p_offset);

	// call the entry point from the ELF header
	// note: does not return!
	((void (*)(void)) (ELFHDR->e_entry))();

bad:
	outw(0x8A00, 0x8A00);
	outw(0x8A00, 0x8E00);
	while (1)
		/* do nothing */;
}
```

这一段不好理解。bootmain函数先定义了两个代码段头文件，见注释。

之后调用readseg从磁盘调一个elf文件到内存，检验如果文件有效，就继续将其他内核段调入内存。elf结构体内容见上文的源文件，最后结果就是这段for循环将内核程序全部从磁盘加载到内存。

下面的bad标号是如果检测elf文件无效，就调用这段代码处理。

主文件部分这就结束了，关键的readseg函数和readsect函数有些点我理解不够，不说了，只需要知道这两个函数的作用就行了。





