*Concepts you may want to Google beforehand: hard disk, cylinder, head, sector, 
carry bit*

**Goal: 使bootsector从磁盘中加载数据来引导内核**

我们的操作系统无法容纳512字节的引导扇区，因此我们需要从磁盘读取数据才能运行内核。
值得庆幸的是，我们不必处理打开和关闭旋转的盘片的问题,我们可以像在屏幕上打印字符一样调用一些BIOS routines。
对此，我们需要将'al'设置为'0x02'**根据下面的指导以及实际代码应该是把ah设为0x02**（以及其他所需带有柱面信息，头部信息，扇面信息的寄存器）并且发出'int 0x13'中断信号

You can access [a detailed int 13h guide here](http://stanislavs.org/helppc/int_13-2.html)

本节课程中我们将第一次使用'carry bit 进位'，该bit是每个寄存器都有的额外bit，用来表示该寄存器存储了超过其容量的信息
```nasm
mov ax, 0xFFFF
add ax, 1 ; ax = 0x0000 and carry = 1
```

该进位无法直接访问，但是可以被其他运算符当作控制结构来使用。例如jc（跳转，如果carry bit = 1）

BIOS同时会将'al'设置为读取的扇区数，所以始终保持将其与期望的数值进行比对


Code
----

Open and examine `boot_sect_disk.asm` for the complete routine that
reads from disk.

`boot_sect_main.asm`为磁盘读取准备了参数，然后调用'disk_load'。注意我们如何写一些实际上不属于引导扇区的额外数据，因为它们在512位标记之外。

boot sector实际上在磁盘0 磁头为0 柱面为0 扇面为1（扇面是从1开始的）的位置

所以任何在512字节之后的数据都对应磁盘0 磁头0 柱面0 扇区0

The main routine will fill it with sample data and then let the bootsector
read it.

**Note: if you keep getting errors and your code seems fine, make sure that qemu
is booting from the right drive and set the drive on `dl` accordingly**

The BIOS sets `dl` to the drive number before calling the bootloader. However,
I found some problems with qemu when booting from the hdd.

There are two quick options:

1. Try the flag `-fda` for example, `qemu -fda boot_sect_main.bin` which will set `dl`
as `0x00`, it seems to work fine then.
2. Explicitly use the flag `-boot`, e.g. `qemu boot_sect_main.bin -boot c` which 
automatically sets `dl` as `0x80` and lets the bootloader read data
