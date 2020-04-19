# exercise 1.3

Q1：

在哪一点处理器开始转为32位模式？什么导致了从16位转为32位？

A1：

boot.s中的下面这段代码。当cr0 register写入1时，就转为了32位。

```assembly
  lgdt    gdtdesc
  movl    %cr0, %eax
  orl     $CR0_PE_ON, %eax
  movl    %eax, %cr0
```



Q2：

boot loader最后执行的是哪一条指令？kernel刚刚加载时执行的是哪一条指令？

A2：

boot loader最后执行的是

```C
	((void (*)(void)) (ELFHDR->e_entry))();
```

kernel开头执行的是

```
f010000c:	66 c7 05 72 04 00 00 	movw   $0x1234,0x472
```



Q3：

为了将整个内核从磁盘加载到内存，boot loader如何决定要加载多少个扇区？它从哪里找到这些信息？

A3：

在一开始读入的elf头文件中，包括了kernel在磁盘中的位置，长度等信息，根据这些信息设置参数，调用for循环和文件接口，就能知道要调入多少扇区。

























