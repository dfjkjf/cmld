---
sort: 20
---

# 华为OD机考

华为OD机考采用ACM模式，即自己管理输入输出，因此必须用好一个类和一个函数
- **streamstring**
- **getline**


## 1. streamstring

`std::stringstream`是 C++标准库中的一个类，它非常有用，主要有以下几个方面的用途：

**一、字符串与其他数据类型的转换**

1. 将其他数据类型转换为字符串：
   - 可以方便地将整数、浮点数等基本数据类型转换为字符串。例如：
   ```cpp
   int num = 42;
   std::stringstream ss;
   ss << num;
   std::string str = ss.str();
   ```
   - 对于复杂的数据结构，如结构体或类的对象，可以通过重载`<<`运算符来实现向`stringstream`的输出，从而转换为字符串表示。

2. 将字符串转换为其他数据类型：
   - 可以从字符串中提取出特定类型的数据。例如：
   ```cpp
   std::string str = "42";
   std::stringstream ss(str);
   int num;
   ss >> num;
   ```

**二、字符串的拼接与格式化**

1. 类似于`sprintf`的功能，但更安全和灵活：
   - 可以将不同类型的数据拼接成一个字符串，并且可以进行格式化输出。例如：
   ```cpp
   int a = 10;
   double b = 3.14;
   std::stringstream ss;
   ss << "a = " << a << ", b = " << b;
   std::string result = ss.str();
   ```

**三、字符串的解析与提取**

1. 从复杂的字符串中提取特定部分：
   - 当有一个包含多种数据的字符串时，可以使用`stringstream`逐步提取出需要的数据。例如，从一个格式为“x:y:z”的字符串中提取出整数`x`、`y`和`z`。
   ```cpp
   std::string str = "10:20:30";
   std::stringstream ss(str);
   int x, y, z;
   char delimiter;
   ss >> x >> delimiter >> y >> delimiter >> z;
   ```

2. 处理复杂的输入格式：
   - 对于不规范的输入，可以使用`stringstream`进行清理和解析。例如，去除输入中的多余空格、特定字符等，然后提取出有效数据。

## 2. getline

`std::getline`主要有以下两种常见用法：

**一、从输入流中读取一行到字符串**

```cpp
std::string str;
std::cout << "Enter a line: ";
std::getline(std::cin, str);
std::cout << "You entered: " << str << std::endl;
```

在这个例子中，`std::getline(std::cin, str)`从标准输入流`std::cin`中读取一行内容，并存储到字符串`str`中。

**二、从其他输入流中读取一行到字符串**

可以从文件输入流或字符串输入流中读取一行。

```cpp
std::istringstream iss("Line 1\nLine 2\nLine 3");
std::string str;
while (std::getline(iss, str)) {
    std::cout << "Read: " << str << std::endl;
}
```

这里从`std::istringstream`对象中逐行读取内容到`str`中，并输出读取到的每一行。

此外，`std::getline`还可以接受第三个参数作为分隔符，默认情况下分隔符是换行符`'\n'`。例如，可以用其他字符作为分隔符来读取字符串：

```
std::string str = "Hello|World|How|Are|You";
std::istringstream iss(str);
std::string token;
while (std::getline(iss, token, '|')) {
    std::cout << "Token: " << token << std::endl;
}
```

在这个例子中，使用`'|'`作为分隔符从字符串中读取各个部分。
