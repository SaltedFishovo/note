

```C
#include <stdio.h>

int main(){

  int result, St_Val = 0,ds ;
  asm ( "movl %%esp,%%ecx\n\t"
        "pushl %%ds\n\t"
        "subl %%esp,%%ecx\n\t"
        "movl %%ecx, %0\n\t"
        "movl (%%esp), %1\n\t"
        "movl %%ds, %2\n\t"
        :"=r"(result), "=r"(St_Val), "=r"(ds)
        :
        :"%ecx" );

        printf("New esp - old esp = %d , ds = %X, stack top = %x", result, ds, St_Val);
}


>./a.out
>New esp - old esp = 4 , ds = 2B, stack top = 804002b
```



PUSH与POP对**段寄存器进行操作**：

- 指令后缀为 *l* 时，会对 *ESP* 进行四字节的增减
  - pushl 会将 ESP - 4，段寄存器的值存至栈顶指针一个双字的**低2字节**处，而高2字节处的数据则是原来的或至0（与具体处理器有关）。
  - popl 的操作与之正好对应，将栈顶指针处双字的低2字节的数据取出，作为段寄存器的值。
- 指令后缀为 *w* 时，对 *ESP* 进行2字节的增减。

 