# exercise 1.7

**要求**：使用QEMU和GDB追踪JOS内核并停在 movl %eax,%cr0 处。检查0x00100000和0xf0100000处的值。之后单步运行这条指令，再重新检查这两处的值。确保你知道发生了什么。

在新的映射机制建立后，如果映射没有就位，哪一条指令会是第一条无法工作的指令？注释掉 movl %eax,%cr0 ，看看你是不是正确的。

----

注意实验文档上所说的，硬件实现的页表转换机制将0xf0000000等那些f打头的16进制地址转到0x00100000。**GDB调试设置断点时，设置的是物理地址，不是逻辑地址**，所以断点设置为kernel的入口地址。

```
b *0x10000c
```

不知为何，断点设置到0x100000不行，可能是因为代码段中那一段标号和段标识我不认识。

0x100000处的反汇编代码如下

```assembly
.globl entry
entry:
	movw	$0x1234,0x472			# warm boot
f0100000:	02 b0 ad 1b 00 00    	add    0x1bad(%eax),%dh
f0100006:	00 00                	add    %al,(%eax)
f0100008:	fe 4f 52             	decb   0x52(%edi)
f010000b:	e4                   	.byte 0xe4

f010000c <entry>:
f010000c:	66 c7 05 72 04 00 00 	movw   $0x1234,0x472
```



当执行到movl %eax,%cr0 时，停下，此时查看两处内存结果如下。

```bash
=> 0x100025:    mov    %eax,%cr0
0x00100025 in ?? ()
(gdb) x/1x 0x00100000
0x100000:       0x1badb002
(gdb) x/1x 0xf0100000
0xf0100000 <_start+4026531828>: 0x00000000
(gdb) 
```

因为0xf0100000处不是我们真正装载内核的地方，逻辑地址0xf0100000被映射成了0x00100000，所以低地址处有内容，高地址处无内容。



当单步执行完movl %eax,%cr0 时，停下，此时查看两处内存结果如下。

```bash
=> 0x100028:    mov    $0xf010002f,%eax
0x00100028 in ?? ()
(gdb) x/1x 0x00100000
0x100000:       0x1badb002
(gdb) x/1x 0xf0100000
0xf0100000 <_start+4026531828>: 0x1badb002
(gdb) 
```

可以看到高地址处和低地址处值相同了。

原因其实在实验指导书里写着。

> Once CR0_PG is set, memory references are virtual addresses that get translated by the virtual memory hardware to physical addresses. entry_pgdir translates virtual addresses in the range 0xf0000000 through 0xf0400000 to physical addresses 0x00000000 through 0x00400000, as well as virtual addresses 0x00000000 through 0x00400000 to physical addresses 0x00000000 through 0x00400000. 

首先明确cr0是什么。cr0全称是control register 0.下面是wiki中的解释。

> The CR0 register is 32 bits long on the [386](https://en.wikipedia.org/wiki/Intel_80386) and higher processors. On [x86-64](https://en.wikipedia.org/wiki/X86-64) processors in [long mode](https://en.wikipedia.org/wiki/Long_mode), it (and the other control registers) is 64 bits long. CR0 has various control flags that modify the basic operation of the processor.

| Bit  | Name |                        Full Name                         |                         Description                          |
| :--: | :--: | :------------------------------------------------------: | :----------------------------------------------------------: |
|  0   |  PE  |                  Protected Mode Enable                   | If 1, system is in [protected mode](https://en.wikipedia.org/wiki/Protected_mode), else system is in [real mode](https://en.wikipedia.org/wiki/Real_mode) |
|  1   |  MP  |                   Monitor co-processor                   | Controls interaction of WAIT/FWAIT instructions with TS flag in CR0 |
|  2   |  EM  |                        Emulation                         | If set, no x87 [floating-point unit](https://en.wikipedia.org/wiki/Floating-point_unit) present, if clear, x87 FPU present |
|  3   |  TS  |                      Task switched                       | Allows saving x87 task context upon a task switch only after x87 instruction used |
|  4   |  ET  |                      Extension type                      | On the 386, it allowed to specify whether the external math coprocessor was an [80287](https://en.wikipedia.org/wiki/80287) or [80387](https://en.wikipedia.org/wiki/80387) |
|  5   |  NE  |                      Numeric error                       | Enable internal [x87](https://en.wikipedia.org/wiki/X87) floating point error reporting when set, else enables PC style x87 error detection |
|  16  |  WP  |                      Write protect                       | When set, the CPU can't write to read-only pages when privilege level is 0 |
|  18  |  AM  |                      Alignment mask                      | Alignment check enabled if AM set, AC flag (in [EFLAGS](https://en.wikipedia.org/wiki/FLAGS_register) register) set, and privilege level is 3 |
|  29  |  NW  |                    Not-write through                     | Globally enables/disable [write-through caching](https://en.wikipedia.org/wiki/Write_through_cache) |
|  30  |  CD  | [Cache](https://en.wikipedia.org/wiki/CPU_cache) disable |          Globally enables/disable the memory cache           |
|  31  |  PG  |                          Paging                          | If 1, enable [paging](https://en.wikipedia.org/wiki/Paging) and use the [§ CR3](https://en.wikipedia.org/wiki/Control_register#CR3) register, else disable paging. |

把eax赋给cr0时，eax=0x80110001，对应上面的标志位就能知道发出了什么控制信息。最关键的是PG，这个信号打开了页表机制，以后都会自动将 0xf0000000 到 0xf0400000 的虚拟（逻辑）地址转成 0x00000000 到 0x00400000 的物理地址。

所以此处会自动把0xf0100000转换成0x00100000，所以两者的值相等。



如果映射机制失败，我觉得 jmp *%eax 之后会失败。因为此时eax的值是0xf010002f,如果没有地址映射，那会指向这个物理高地址，而不是本应指向的0x100000附近的低地址，就会出错。

```bash
=> 0x100025:    mov    $0xf010002c,%eax
0x00100025 in ?? ()
(gdb) 
=> 0x10002a:    jmp    *%eax
0x0010002a in ?? ()
(gdb) 
=> 0xf010002c <relocated>:      add    %al,(%eax)
relocated () at kern/entry.S:74
74              movl    $0x0,%ebp                       # nuke frame pointer
(gdb) 
Remote connection closed
(gdb) 
```

上面是注释掉  movl %eax,%cr0 之后的调试结果。果然，跳转之后的第一条指令就报错了。