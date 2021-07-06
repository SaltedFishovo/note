### 函数指针

##### 定义

```c 
//最基本的定义方法
retval_type (*pFun) (paral_t ,paral_t..paral3);
//定义了一个函数指针pFun，指向的函数的返回值类型为int，具有两个整型参数
int (* pFun) (int,int);


//定义了一个函数指针的类型，这个函数指针类型tpFun可以用于声明\定义具有如下返回值，参数的函数指针。
typedef retval_type (*tpFun) (paral_t,paral_t..,paral_t);
```

##### 初始化

```c
int Fun (int, int){
    ...
}
or
int Fun(int,int);

//有两种初始化方法
pFun = Fun;
pFun = &Fun;
```

- 函数名在被使用时，编译器总是会将其**隐式转换为一个函数指针**。&只是说明了编译器的一种隐式操作。
- 在函数指针初始化前，有该函数的原型是必须的，如果没有原型，编译器不知道函数的类型和函数指针所指向的函数类型是否一致。

##### 调用

```c
//调用一个函数指针指向的函数有以下两种方法：
*pFun(1,2);
 pFun(1,2);
```

##### 作为参数

```c
//某个函数的参数是一个函数指针
int aaa(int (* pf) (int,int)){
    ...
}

int (* pFun) (int,int) = Fun;
//调用该函数
aaa(pFun);
aaa(&Fun);
aaa(Fun);
```



### 关键字:inline volatile register

volatile-register：

- volatile所声明的变量不会被catch、register缓存。
- register声明定义的变量存放于寄存器中。
- 

```c
char *ch;
while ((ch = *(unsigned char *) fmt++) != '%') 
```

1. 其中fmt为指向字符数组的指针，即：*char \* fmt* 。若 *fmt++* ，那是否能判断该数组中第一个字符为 %。
2. 为什么需要有强制类型转换：*(unsigned char \*)*。

### 在x.h中 有extern int x；在x.c中定义了全局变量int x，且x.c 引用了x.h，这样做的结果。

### 两个指针相减