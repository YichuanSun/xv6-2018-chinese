# exercise 1.5

**要求**：如果链接地址是错的，追踪boot loader代码会做什么。将boot/MakeFrag文件中的链接地址改成一个错误值，重新编译，追踪boot loader执行，看看会发生什么。不要忘记做完后将正确的链接地址改回去。

---

我把链接地址改成了0x7cc0，结过qemu窗口出现了如下错误信息。

```terminal
qemu-system-i386 -drive file=obj/kern/kernel.img,index=0,media=disk,format=raw -serial mon:stdio -gdb tcp::26000 -D qemu.log  -S
VNC server running on `127.0.0.1:5900'
EAX=00000011 EBX=00000000 ECX=00000000 EDX=00000080
ESI=00000000 EDI=00000000 EBP=00000000 ESP=00006f20
EIP=00007c2d EFL=00000006 [-----P-] CPL=0 II=0 A20=1 SMM=0 HLT=0
ES =0000 00000000 0000ffff 00009300 DPL=0 DS16 [-WA]
CS =0000 00000000 0000ffff 00009b00 DPL=0 CS16 [-RA]
SS =0000 00000000 0000ffff 00009300 DPL=0 DS16 [-WA]
DS =0000 00000000 0000ffff 00009300 DPL=0 DS16 [-WA]
FS =0000 00000000 0000ffff 00009300 DPL=0 DS16 [-WA]
GS =0000 00000000 0000ffff 00009300 DPL=0 DS16 [-WA]
LDT=0000 00000000 0000ffff 00008200 DPL=0 LDT
TR =0000 00000000 0000ffff 00008b00 DPL=0 TSS32-busy
GDT=     00ffb1e8 00000001
IDT=     00000000 000003ff
CR0=00000011 CR2=00000000 CR3=00000000 CR4=00000000
DR0=00000000 DR1=00000000 DR2=00000000 DR3=00000000 
DR6=ffff0ff0 DR7=00000400
EFER=0000000000000000
Triple fault.  Halting for inspection via QEMU monitor.
```

对应的，gdb调试窗口循环执行这条语句。

```terminal
[   0:7c2d] => 0x7c2d:  ljmp   $0x8,$0x7cf2
0x00007c2d in ?? ()
(gdb) 
```

所以停在了 ljmp   $0x8,$0x7cf2 这条语句上，位于boot.s line 55.











