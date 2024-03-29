 # Bootloader

&emsp;&emsp;简单来说，Bootloader就是一段加载操作系统的代码。从功能上，Bootloader可分为两个阶段：

- 第一个阶段初始化一些必要的处理器状态
- 第二个阶段从某个存储设备加载操作系统内核到内存

&emsp;&emsp;Bootloader可以汇编语言编写，也可以用汇编语言结合C语言进行编写。但是使用纯汇编语言编写一个稍微完整一点的Bootloader，会导致代码长度巨大，较难读懂，所以使用汇编语言结合C语言编写是一个很好的选择。

&emsp;&emsp;Bootloader第一阶段需要初始化处理器的状态，大量设置寄存器，完成一些C语言无法直接实现的操作，所以用汇编语言编写是最佳的选择。例如需要设置CR0来进入保护模式、设置栈指针等，这些只需要一条指令便可实现，而在C语言中是没有办法直接实现的。

> 此处需描述x86架构处理器启动过程

&emsp;&emsp;BIOS最后会根据用户设置的默认启动设备的第一个扇区的内容加载至内存0x7c00处，同时检测最后两个字节是否为0x55，0xAA，以判断该程序是一个有效的Bootloader。

&emsp;&emsp;如果是，那么BIOS就认为这个扇区内是一个Bootloader，就将控制转移到这个Bootloader，即在0x7c00处执行指令。

&emsp;&emsp;现在细化的讲一下Bootloader的功能，其第一阶段的工作流程是这样的：**打开A20地址线、加载GDTR、开启保护模式、加载段选择器、控制转移至第二阶段代码**。

> 此处对上述某几个步骤做出解释及实现方法



