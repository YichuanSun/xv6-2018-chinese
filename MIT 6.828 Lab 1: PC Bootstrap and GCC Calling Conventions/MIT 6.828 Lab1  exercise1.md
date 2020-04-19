#  exercise 1.1

要求熟悉一下6.828这门课的参考文献，为以后阅读和写汇编代码打基础。

注意特别提及的那片参考文献，讲解了两种汇编语言格式的不同之处。下面是这篇引文的小总结。



AT&T格式语法采用了一种独特的内联汇编技巧，下面是AT&T格式与Intel格式的不同之处。

- 编译器命名：采用前缀%

  - AT&T：%eax
  - Intel：eax

- 源操作数/目的操作数顺序：AT&T格式中源操作数总是在左边，目的操作数总是在右边。

  下面是将eax加载进ebx的例子。

  - AT&T：movl %eax, %ebx
  - Intel：mov ebx, eax

- 常量/立即数格式：所有的立即数、常量必须加前缀$

  下面的例子是将C变量booga的地址加载到eax。这个地址是静态的。

  - AT&T：movl $_booga, %eax
  - Intel：mov eax, _booga

- 操作符格式要求：必须用后缀b，w，或者l来明确目的寄存器的数据宽度，它们分别表示宽度为byte，word，或longword。如果不这样做，汇编语言编译器会猜。Intel格式中也有对应的方式，不说了。

- 引用内存：下面是标准的32为地址格式

  - AT&T：immed32( basepointer, indexpointer, indexscale )
  - Intel：[ basepointer + indexpointer * indexscale + immed32 ]

  它们相当与计算的内存位置为immed32+basepointer+indexpointer*indexscale

  其他简单的内存定址格式：

  ​		定位特定C变量

  - AT&T：_booga

  - Intel：[_booga]

    下划线"\_"代表这是一个静态（全局）C变量。由于汇编和C是一起编译的，Unix和C语言某些变量名可能冲突，所以为了防止类似的符号名冲突，UNIX下的C语言就规定，C语言的源代码文件中的所有全局变量和函数经过编译后，相应的符号名前面会自动的加上下划线。这样做的好处，就是方便是程序开发人员，不用太小心翼翼的起名，避免了与汇编文件中的符号名的冲突。

    定位寄存器的指向

  - AT&T：(%eax)

  - Intel：[eax]

















