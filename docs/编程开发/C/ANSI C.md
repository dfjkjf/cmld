---
sort: 1
---

# ANSI C

## 关键字

```c
    #define   // 宏定义命名，定义一个标识符来表示一个常量
    #include  // 文件包含命令，用来引入对应的头文件或其他文件
    #undef    // 来将前面定义的宏标识符取消定义
    #ifdef    // 条件编译
    #ifndef   // 条件编译
    #if       // 条件编译
    #else     // 条件编译
    #elif     // 条件编译
    #endif    // 条件编译
    #error    // 用于生成一个编译错误消息

    __DATE__  // 当前日期，一个以 “MMM DD YYYY” 格式表示的字符串常量
    __TIME__  // 当前时间，一个以 “HH:MM:SS” 格式表示的字符串常量。
    __FILE__  // 这会包含当前文件名，一个字符串常量。
    __LINE__  // 这会包含当前行号，一个十进制常量。
    __STDC__  // 当编译器以 ANSI 标准编译时，则定义为 1；判断该文件是不是标准C程序。

```

```c
#include <stdio.h>

int main()
{
    printf("Hello World!\n");
    printf("%s\n", __FILE__);
    printf("%d\n", __LINE__);
    printf("%s\n", __DATE__);
    printf("%d\n", __TIME__);
    printf("%d\n", __STDC__);
    printf("%d\n", MyName);

    return 0;
}
```

