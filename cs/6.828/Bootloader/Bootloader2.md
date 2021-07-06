# Bootloader2

&emsp;&emsp;第二阶段的主要工作就是从某个存储设备读取的指定位置读取操作系统至内存，并将处理器控制权转交给操作系统内核。在xv6的Bootloader里，我们假设内核所在的存储设备和在存储设备上的位置是已知的。

> 位置

&emsp;&emsp;其工作流程是这样的：**先加载内核首部ELF头若干扇区、判断是否为有效的ELF文件、通过程序头部表将整个内核加载至内存、转移控制权给内核。**

```c++
#define ELFHDR	((struct Elf *) 0x10000) 
```

&emsp;&emsp;首先在该文件中定义了一个宏 *ELFHDR* ，*(struct Elf \*) 0x10000* 是为了将0x10000看作一个指向 *Elf头部* 结构体的指针，即该地址处存放着一个Elf的结构体，这样，*((struct Elf \*) 0x10000) ->* XX 或者*ELFHDR* >* XX 便可直接引用这个结构体的成员。

```c
readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);
```

  *readseg*从磁盘上读取了8个扇区大小的数据至内存 *0x10000*，这个位置

> ...0位置的来源

&emsp;&emsp;程序头部表（Program Header Table）的位置存在于Elf头部后面，程序头部表将整个Elf文件划分为若干个段（Segment），描述了Elf文件的段从文件内到内存中的映射关系，即某一个段需要加载至内存什么位置。它的结构就是一个结构体的数组，数组中的每一个元素就是一个描述段的结构体。其中  *(struct Proghdr \*)* 就指向一个描述段的结构体。

```c
ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);
```

&emsp;&emsp;*ELFHDR->e_phoff* 成员存放了程序头部表在Elf文件中的以字节为单位的偏移量，因为 *ELFHDR* 的类型为指向Elf结构体的指针，如果直接相加，得到的地址会这样计算：*0x10000 + 程序头表的偏移量 x Elf结构体的大小*。所以需要将ELFHDR的类型强制转换为指向字节类型的指针，才能得到的正确地址。

&emsp;&emsp;然后将这个地址强制转换为指向Proghdr结构体的指针，以便于之后的对程序头的引用。

```c
 eph = ph + ELFHDR->e_phnum;
```

&emsp;&emsp;*ELFHDR->e_phnum* 成员保存了内核需要加载进内存的段的个数，此时*ph*指向第一个段，此处表达式就是一个指针加法，值为最后一个段结构体的下一个字节，用来为循环读取段的结构体作为结束判断条件。

```C
	for (; ph < eph; ph++)
		readseg(ph->p_pa, ph->p_memsz, ph->p_offset);
```

*p_pa* 中的 *pa* 全称为 *physical address* ，在jos的内核中，这个成员存放了该段要加载的物理地址，可以由链接脚本命令 *AT* 在链接时明确指定。*p_memsz* 成员就是该段在内存中的大小， *p_offset* 成员指明了该段在内核中的偏移。所以这个精简的循环通过遍历程序头部表来加载整个内核。

> 是否有页对齐+内核内存结构视图

```C
((void (*)(void)) (ELFHDR->e_entry))();
```

&emsp;&emsp;*ELFHDR->e_entry* 指向了内核的入口点，即该代码首先将*ELFHDR->e_entry*转换为一个返回值为void，参数为void的函数指针。因为函数指针可以被调用，其格式与调用函数一样，所以该代码随后直接调用这个函数。

