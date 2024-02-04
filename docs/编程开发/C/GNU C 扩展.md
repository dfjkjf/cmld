---
sort: 2
---


# GNU C 扩展

| 语法 | 作用 |
| --- | --- |
| 指定初始化 | 支持指定任意元素初始化，不再按照固定的顺序初始化 |
| 语句表达式 | 在一个表达式里内嵌语句，使用局部变量、for 循环和 goto 跳转语句 |
| 零长度数组 | 长度为0的数组，通过作为变长结构体的成员 |
| 变参函数 | 长度为0的数组，通过作为变长结构体的成员 |
| | |
| **关键字** |  |
| `typeof` | 获取一个变量或表达式的类型 |
| `__alignof__` | 获取一个变量或表达式的内存对齐量 |
| | |
| **宏** |  |
| `offsetof(TYPE, MEMBER)` | 计算结构体某一成员在结构体内的偏移 |
| `container_of(ptr, type, member)` | 根据结构体某一成员的地址，获取这个结构体的首地址 |
| | |
| **\_\_atttribute\_\_((ATTRIBUTE))** | 声明一个函数、变量或类型的特殊属性，指导编译器在编译程序时进行特定方面的优化或代码检查 |
| `section` | 将一个函数或变量放到指定的段 |
| `aligned` | 指定一个变量或类型的对齐方式，一般用来增大变量的地址对齐 |
| `packed` | 指定一个变量或类型的对齐方式，使用最可能小的地址对齐方式 |
| `format` | 指定变参函数的参数格式检查 |
| `weak` | 将一个强符号转换为弱符号,适用全局变量或全局函数 |
| `alias` | 给函数定义一个别名 |
| `noinline` | 指定的函数内联不展开 |
| `always_inline` | 指定的函数内联展开 |

## 指定初始化

```c
/* 初始化数组 */
int b[100] ={ [10] = 1, [11 ... 30] = 10, [50 ... 60] = 2， [90] = 2};

switch(i)
    {
        case 1:
            printf("1\n");
            break;
        case 2 ... 8:
            printf("%d\n",i);
            break;
        case 9:
            printf("9\n");
            break;
        default:
            printf("default!\n");
            break;
    }
    
/* 初始化结构体 */
void my_func(){ ... }

struct student{
    char name[32];
    int age;
    void (*func)();
};
    struct student stu2=
    {
        .name = get_name,
        .func = my_func,
        .age  = 18
    };
```

## 语句表达式

语句表达式最外面使用小括号()括起来，里面一对大括号{}包起来的是代码块，代码块里允许内嵌各种语句。语句的格式可以是 “表达式;”这种一般格式的语句，也可以是循环、跳转等语句。

```c
({ 表达式1; 表达式2; 表达式3; })
```

语句表达式的值总等于最后一个表达式的值。
```c
sum = 
    ({
        int s = 0;
        for( int i = 0; i < 10; i++)
            s = s + i;
            s;
    });
```

## 零长度数组

```c
int len;
int a[len]; // 变长数组

int b[0];   // 零长度数组，sizeof(b) == 0
```

它不占用内存存储空间，仅是符号，当数组首地址使用。

它常常作为结构体的一个成员，构成一个变长结构体

```c
struct buffer{
    int len;
    int a[0];
};

struct buffer *buf = (struct buffer *)malloc(sizeof(struct buffer)+ 20);
buf->len = 20;
strcpy(buf->a, "hello world!\n");
```

## typeof关键字

- 例子：
```c
int i ;
typeof(i) j = 2; // int j = 2;

typeof(int *) a; // int *a;

int func();
typeof(f()) k; // int k;

#define MAX(x,y)             \
({                           \
    typeof(x) _x = x;        \
    typeof(x) _y = y;        \
    (void) (&_x == &_y);     /* warning：comparison of distinct pointer types lacks a cast */ \
    _x > _y ? _x : _y;       \
})
```

## __alignof__关键字

- 例子：
```c
struct ship
{
    int year_built;
    char canons;
    int mast_height;
}__attribute__((aligned(4)));
struct ship2
{
    int year_built;
    char canons __attribute__((aligned(4)));
    int mast_height;
};

printf ("%lu\n", __alignof__(ship.canons));  // 1
printf ("%lu\n", __alignof__(ship2.canons)); // 4

```

## offsetof

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

## container_of

```c
#define  container_of(ptr, type, member) ({    \
     const typeof( ((type *)0)->member ) *__mptr = (ptr); \
     (type *)( (char *)__mptr - offsetof(type,member) ); \
})
```
- 例子：
```c
struct student
{
    char *name;
    int num;
    int age;
};
int main(void)
{
    struct student stu;
    struct student *p;
    
    p = container_of( &stu.num, struct student, num);
    return 0;
}
```

## atttribute

声明属性：
```c
section // 将一个函数或变量放到指定的段
aligned // 显式指定一个变量的存储边界对齐方式，一般用来增大变量的地址对齐
packed  // 显式指定一个变量的存储边界对齐方式，使用最可能小的地址对齐方式
format  // 指定变参函数的参数格式检查
weak    // 将一个强符号转换为弱符号
alias   // 给函数定义一个别名
noinline      // 指定的函数内联不展开
always_inline // 指定的函数内联展开
……
```
**属性声明要紧挨着变量**

### aligned & packed
```c
int uninit_val __attribute__((section(".data")));

char c1 __attribute__((aligned(8)) = 4;
char c2 __attribute__((packed,aligned(4)));
__attribute__((packed,aligned(4))) char c2 = 4;
char c2 = 4 __attribute__((packed,aligned(4)));  // error

// size: 8
struct data{
    char a;
    short b;
    int c ;
};

// size: 12
struct data{
    char a;
    short b __attribute__((aligned(4)));
    int c ;
};

// size: 16
struct data{
    char a;
    short b;
    int c ;
}__attribute__((aligned(16)));

// size: 7
struct data{
    char a;
    short b __attribute__((packed));
    int c __attribute__((packed));
};
struct data{
    char a;
    short b;
    int c ;
}__attribute__((packed));

// 避免了结构体内因地址对齐产生的内存空洞，又指定了整个结构体的对齐方式
struct data{
    char a;
    short b;
    int c ;
}__attribute__((packed, aligned(8))));
```
### format
```c
__attribute__(( format (archetype, string-index, first-to-check)))
void LOG(const char *fmt, ...)  __attribute__((format(printf,1,2)));
void LOG2(int num, char *fmt, ...)  __attribute__((format(printf,2,3)));
```
属性 format(printf,1,2) 有三个参数。第一个参数 printf 是告诉编译器，按照 printf 函数的检查标准来检查；第2个参数表示在 LOG 函数所有的参数列表中，格式字符串的位置索引；第3个参数是告诉编译器要检查的参数的起始位置。

```c
LOG("I am li\n")；
LOG("I am li, I have %d houses!\n", num);
LOG("I am li, I have %d houses! %d cars\n", num1, num2);
LOG(num, "I am li, I have %d houses! %d cars\n", num1, num2);
```

### weak
```c
//func.c
int a __attribute__((weak)) = 1;
void func(void)
{
    printf("func：a = %d\n", a);
}
//main.c
int a = 4;
void func(void);
int main(void)
{
    printf("main：a = %d\n", a);
    func();
    return 0;
}
运行结果:
main: a = 4
func: a = 4

//func.c
int a __attribute__((weak)) = 1;
void __attribute__((weak)) func(void)
{
    printf("func：a = %d\n", a);
}
//main.c
int a = 4;
void func(void)
{
    printf("I am a strong symbol!\n");
}
int main(void)
{
    printf("main：a = %d\n", a);
    func();
    return 0;
}
运行结果:
main: a = 4
func: I am a strong symbol!
```
**弱符号的用途**

当函数被声明为一个弱符号时：当链接器找不到这个函数的定义时，也不会报错。编译器会将这个函数名，即弱符号，设置为0或一个特殊的值。只有当程序运行时，调用到这个函数，跳转到0地址或一个特殊的地址才会报错。

```c
//func.c
int a __attribute__((weak)) = 1;

//main.c
int a = 4;
void __attribute__((weak)) func(void);
int main(void)
{
    printf("main：a = %d\n", a);
    func();
    return 0;
}

运行结果:
main: a = 4
Segmentation fault (core dumped)
```

为了防止函数运行出错，我们可以在运行这个函数之前，先做一个判断，即看这个函数名的地址是不是0，然后再决定是否调用。这样就可以避免段错误了。

```c
//func.c
int a __attribute__((weak)) = 1;

//main.c
int a = 4;
void __attribute__((weak)) func(void);
int main(void)
{
    printf("main：a = %d\n", a);
    if (func)
        func();
    return 0;
}
运行结果:
main: a = 4
```

弱符号的这个特性，在库函数中应用很广泛。比如你在开发一个库，基础的功能已经实现，有些高级的功能还没实现，那你可以将这些函数通过 weak 属性声明，转换为一个弱符号。通过这样设置，即使函数还没有定义，我们在应用程序中只要做一个非0的判断就可以了，并不影响我们程序的运行。等以后你发布新的库版本，实现了这些高级功能，应用程序也不需要任何修改，直接运行就可以调用这些高级功能。

弱符号还有一个好处，如果我们对库函数的实现不满意，我们可以自定义与库函数同名的函数，实现更好的功能。比如我们 C 标准库中定义的 gets() 函数，就存在漏洞，常常成为黑客堆栈溢出攻击的靶子。

```c
int main(void)
{
    char a[10];
    gets(a);
    puts(a);
    return 0;   
}
```

C 标准定义的库函数 gets() 主要用于输入字符串，它的一个 Bug 就是使用回车符来判断用户输入结束标志。这样的设计很容易造成堆栈溢出。比如上面的程序，我们定义一个长度为10的字符数组用来存储用户输入的字符串，当我们输入一个长度大于10的字符串时，就会发生内存错误。

接着我们定义一个跟 gets() 相同类型的同名函数，并在 main 函数中直接调用，代码如下。

```c
#include<stdio.h>

char * gets (char * str)
{
    printf("hello world!\n");
    return (char *)0;
}

int main(void)
{
    char a[10];
    gets(a);
    return 0;   
}
运行结果:
hello world!
```

通过运行结果，我们可以看到，虽然我们定义了跟 C 标准库函数同名的 gets() 函数，但编译是可以通过的。程序运行时调用 gets() 函数时，就会跳转到我们自定义的 gets() 函数中运行。

### alias
```c
void __f(void)
{
    printf("__f\n");
}

void f() __attribute__((alias("__f")));
int main(void)
{
    f();
    return 0;   
}
```

### noinline & always_inline

**内联函数**

有些函数很小，而且调用频繁，调用开销大，算下来性价比不高。我们就可以将这个函数声明为内联函数。编译器在编译过程中遇到内联函数时，像宏一样，将内联函数直接在调用处展开。

好处：
- 减少调用函数时的开销
- 减少传参时可能引起的压栈出栈的开销。
- 减少PC跳转时对流水线的破坏。

坏处：
- 代码所占体积会更大。

**编译器对内联函数的处理**

`inline`关键字只是建议编译器将一个函数声明为内联函数，但编译器不一定会对这个内联函数展开处理。编译器也要进行评估，权衡展开和不展开的利弊。

一般来讲，判断对一个内联函数到底展不展开，从程序员的角度，主要考虑以下几个因素。
-   函数体积小且调用频繁
-   函数体内无递归、循环等语句
-   函数本身当作一个函数指针在别处被引用
-   函数和调用该函数调用者是否在同一文件内

当我们认为一个函数体积小，而且被大量频繁调用，应该做内联展开时，就可以使用 `inline` 关键字修饰它。但编译器会不会作内联展开，编译器也会有自己的权衡。如果你想告诉编译器一定要展开，或者不作展开，就可以使用 `noinline` 或 `always_inline` 对函数作一个属性声明。

```c
//main.c
static inline 
__attribute__((always_inline))  int func(int a)
{
    return a+1;
}
​
static inline void print_num(int a)
{
    printf("%d\n",a);
}
int main(void)
{
    int i;
    i=func(3);
    print_num(10);
    return 0;
}
```

在这个程序中，我们分别定义两个内联函数 func() 和 print_num()，然后使用 `always_inline` 对 func() 函数进行属性声明。接下来，我们对生成的可执行文件 a.out 作反汇编处理，其汇编代码如下。

```c
$ gcc -o a.out main.c
$ objdump -D a.out 
00010438 <print_num>:
   10438:    e92d4800    push    {fp, lr}
   1043c:    e28db004    add fp, sp, #4
   10440:    e24dd008    sub sp, sp, #8
   10444:    e50b0008    str r0, [fp, #-8]
   10448:    e51b1008    ldr r1, [fp, #-8]
   1044c:    e59f000c    ldr r0, [pc, #12]
   10450:    ebffffa2    bl  102e0 <printf@plt>
   10454:    e1a00000    nop ; (mov r0, r0)
   10458:    e24bd004    sub sp, fp, #4
   1045c:    e8bd8800    pop {fp, pc}
   10460:    0001050c    andeq   r0, r1, ip, lsl #10

00010464 <main>:
   10464:    e92d4800    push    {fp, lr}
   10468:    e28db004    add fp, sp, #4
   1046c:    e24dd008    sub sp, sp, #8
   10470:    e3a03003    mov r3, #3
   10474:    e50b3008    str r3, [fp, #-8]
   10478:    e51b3008    ldr r3, [fp, #-8]
   1047c:    e2833001    add r3, r3, #1
   10480:    e50b300c    str r3, [fp, #-12]
   10484:    e3a0000a    mov r0, #10
   10488:    ebffffea    bl  10438 <print_num>
   1048c:    e3a03000    mov r3, #0
   10490:    e1a00003    mov r0, r3
   10494:    e24bd004    sub sp, fp, #4
   10498:    e8bd8800    pop {fp, pc}
```
通过反汇编代码可以看到，因为我们对 func() 函数作了 always_inline 属性声明，所以编译器在编译过程中，对于 main()函数调用 func()，会直接在调用处展开。

```c
   10470:    e3a03003    mov r3, #3
   10474:    e50b3008    str r3, [fp, #-8]
   10478:    e51b3008    ldr r3, [fp, #-8]
   1047c:    e2833001    add r3, r3, #1
   10480:    e50b300c    str r3, [fp, #-12]
```
而对于 `print_num()` 函数，虽然我们对其作了内联声明，但编译器并没有对其作内联展开，而是当作一个普通函数对待。还有一个注意的细节是，当编译器对内联函数作展开处理时，会直接在调用处展开内联函数的代码，不再给 func() 函数本身生成单独的汇编代码。这是因为其它调用该函数的位置都作了内联展开，没必要再去生成。在这个例子中，我们发现就没有给 func() 函数本身生成单独的汇编代码，编译器只给 print_num() 函数生成了独立的汇编代码。


**static & inline**

在 Linux 内核中，你会看到大量的内联函数定义在头文件中，而且常常使用 static 修饰。

内联函数为什么要定义在头文件中呢？
- 因为它是一个内联函数，可以像宏一样使用，任何想使用这个内联函数的源文件，不必亲自再去定义一遍，直接包含这个头文件，即可像宏一样使用。

那为什么还要用 static 修饰呢？
- 因为我们使用 inline 定义的内联函数，编译器不一定会内联展开，那么当多个文件都包含这个内联函数的定义时，编译时就有可能报重定义错误。而使用 static 修饰，可以将这个函数的作用域局限在各自本地文件内，避免了重定义错误。