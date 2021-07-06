### BootLoader 的链接

BootLoader通过 *Boot.o* 和 *main.o* 两个文件链接生成。

```
ld
   -m
   elf_i386
   -N
   -e
   start
   -Ttext
   0x7C00
   -o
   obj/boot/boot.out
   obj/boot/boot.o
   obj/boot/main.o
```

对于参加链接的两个文件 *boot.o、main.o* ，这两个文件在链接时命令行参数给出的顺序是：*boot.o* 必须在 *main.o* 前面。 这会使得在链接好的文件中， *boot.o* 中的代码在 *main.o* 的前面，而 *boot.o* 中存放着启动计算机时先要执行的指令。

我们对上述链接器使用的选项作出解释。

- *-m* 指定此次链接要生成在 *i386* 体系结构下的一个 32 位的ELF文件。使用该选项后，通过设置location counter，可以将第一个Section的 *file offset* 设置为 *0x1000* 。

>       -N
>       --omagic
>           Set the text and data sections to be readable and writable.  Also,
>           do not page-align the data segment, and disable linking against
>           shared libraries.  If the output format supports Unix style magic
>           numbers, mark the output as "OMAGIC". Note: Although a writable
>           text section is allowed for PE-COFF targets, it does not conform to
>           the format specification published by Microsoft.
>           
>           2.1.7 Options specific to PDP11 targets
>    For the pdp11-aout target, three variants of the output format can be produced as selected by the following options. The default variant for pdp11-aout is the ‘--omagic’ option, whereas for other targets ‘--nmagic’ is the default. The ‘--imagic’ option is defined only for the pdp11-aout target, while the others are described here as they apply to the pdp11-aout target.
>    
>    -N
>    --omagic
>    Mark the output as OMAGIC (0407) in the a.out header to indicate that the text segment is not to be write-protected and shared. Since the text and data sections are both readable and writable, the data section is allocated immediately contiguous after the text segment. This is the oldest format for PDP11 executable programs and is the default for ld on PDP11 Unix systems from the beginning through 2.11BSD.

- *-N*  根据手册描述，该选项将目标文件链接为一个PDP11 target 这个目标文件有以下特性（刚好是我们需要的）：
  - 取消了Segment的页面对齐，否则加载需要加载进内存的Segment需要对齐至页面大小。
  -  *.text* 和 *.data* 设置成可读可写的，这使得这两个节会合并成一个Segment。
  - 会使第一个*Section file offset = 0x74* ，并且不受到链接脚本的影响！



>       -e entry
>       --entry=entry
>           Use entry as the explicit symbol for beginning execution of your
>           program, rather than the default entry point.  If there is no
>           symbol named entry, the linker will try to parse entry as a number,
>           and use that as the entry address (the number will be interpreted
>           in base 10; you may use a leading 0x for base 16, or a leading 0
>           for base 8).

- *-e* 设置程序入口点为 *start* ，感觉没什么用。。
- *-Ttext* 设置代码段的虚拟地址为 0x7c00 ，作用等同于在链接脚本中 .text section 前设置 *location conuter*。

### 内核的链接



```
ld
   -o
   obj/kern/kernel
   
   -m
   elf_i386
   
   -T
   kern/kernel.ld
   
   -nostdlib
   
   obj/kern/entry.o
   obj/kern/entrypgdir.o
   obj/kern/init.o
   obj/kern/console.o
   obj/kern/monitor.o
   obj/kern/pmap.o
   obj/kern/kclock.o
   obj/kern/printf.o
   obj/kern/kdebug.o
   obj/kern/printfmt.o
   obj/kern/readline.o
   obj/kern/string.o   
   /usr/lib/gcc/x86_64-linux-gnu/5/32/libgcc.a
   
   -b
   binary

```

- *-nostdlib* 

> -nostdlib 
>
> Only search library directories explicitly specified on the command line.
>
>  Library directories specified in linker scripts (including linker scripts specified on the command line) are ignored.