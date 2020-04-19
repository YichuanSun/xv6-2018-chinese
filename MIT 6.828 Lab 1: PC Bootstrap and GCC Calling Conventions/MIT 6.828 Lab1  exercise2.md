# exercis 1.2

```terminal
The target architecture is assumed to be i8086
[f000:fff0]    0xffff0:	ljmp   $0xf000,$0xe05b
0x0000fff0 in ?? ()
+ symbol-file obj/kern/kernel
(gdb) si
[f000:e05b]    0xfe05b:	cmpl   $0x0,%cs:0x6ac8
0x0000e05b in ?? ()
(gdb) si
[f000:e062]    0xfe062:	jne    0xfd2e1
0x0000e062 in ?? ()
(gdb) si
[f000:e066]    0xfe066:	xor    %dx,%dx
0x0000e066 in ?? ()
(gdb) si
[f000:e068]    0xfe068:	mov    %dx,%ss
0x0000e068 in ?? ()
(gdb) 
[f000:e06a]    0xfe06a:	mov    $0x7000,%esp
0x0000e06a in ?? ()
(gdb) si
[f000:e070]    0xfe070:	mov    $0xf34c2,%edx
0x0000e070 in ?? ()
(gdb) si
[f000:e076]    0xfe076:	jmp    0xfd15c
0x0000e076 in ?? ()
(gdb) si
[f000:d15c]    0xfd15c:	mov    %eax,%ecx
0x0000d15c in ?? ()
(gdb) si
[f000:d15f]    0xfd15f:	cli                     ;屏蔽中断
0x0000d15f in ?? ()
(gdb) si
[f000:d160]    0xfd160:	cld                     ;清方向标志位
0x0000d160 in ?? ()
(gdb) si
[f000:d161]    0xfd161:	mov    $0x8f,%eax
0x0000d161 in ?? ()
(gdb) 
[f000:d167]    0xfd167:	out    %al,$0x70
0x0000d167 in ?? ()
(gdb) si
[f000:d169]    0xfd169:	in     $0x71,%al
0x0000d169 in ?? ()
(gdb) si
[f000:d16b]    0xfd16b:	in     $0x92,%al
0x0000d16b in ?? ()
(gdb) si
[f000:d16d]    0xfd16d:	or     $0x2,%al
0x0000d16d in ?? ()
(gdb) 
[f000:d16f]    0xfd16f:	out    %al,$0x92
0x0000d16f in ?? ()
(gdb) si
[f000:d171]    0xfd171:	lidtw  %cs:0x6ab8
0x0000d171 in ?? ()
(gdb) si
[f000:d177]    0xfd177:	lgdtw  %cs:0x6a74
0x0000d177 in ?? ()
(gdb) si
[f000:d17d]    0xfd17d:	mov    %cr0,%eax
0x0000d17d in ?? ()
(gdb) si
[f000:d180]    0xfd180:	or     $0x1,%eax
0x0000d180 in ?? ()
(gdb) si
[f000:d184]    0xfd184:	mov    %eax,%cr0
0x0000d184 in ?? ()
(gdb) si
[f000:d187]    0xfd187:	ljmpl  $0x8,$0xfd18f        ;跳转到i386模式
0x0000d187 in ?? ()
(gdb) si
The target architecture is assumed to be i386
=> 0xfd18f:	mov    $0x10,%eax
0x000fd18f in ?? ()
(gdb) si
```

















