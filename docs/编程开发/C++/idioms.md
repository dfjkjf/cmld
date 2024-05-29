---
sort: 2
---

# idioms

C++ 习语是 C++ 编程中常用的既定模式或技术，用于实现特定的结果。它们有助于提高代码的效率、可维护性并减少错误。下面是一些常见的 C++ 习语：

1. 资源获取即初始化（RAII）

这个习语将资源的生命周期与对象的生命周期绑定在一起，从而确保资源总是被正确地获取和释放。创建对象时，它会获取资源；销毁对象时，它会释放资源。

```c++
class Resource {
public:
    Resource() { /* Acquire resource */ }
    ~Resource() { /* Release resource */ }
};

void function() {
    Resource r; // Resource is acquired
    // ...
} // Resource is released when r goes out of scope
```

2. 三人规则

如果一个类定义了以下任何一项，它就应该定义所有三项：复制构造函数、复制赋值操作符和析构函数。

```c++
class MyClass {
public:
    MyClass();
    MyClass(const MyClass& other); // Copy constructor
    MyClass& operator=(const MyClass& other); // Copy assignment operator
    ~MyClass(); // Destructor
};
```

3. 五人规则

在 C++11 中，"三规则 "扩展为 "五规则"，包括移动构造函数和移动赋值操作符。

```c++
class MyClass {
public:
    MyClass();
    MyClass(const MyClass& other); // Copy constructor
    MyClass(MyClass&& other); // Move constructor
    MyClass& operator=(const MyClass& other); // Copy assignment operator
    MyClass& operator=(MyClass&& other); // Move assignment operator
    ~MyClass(); // Destructor
};
```

4. 删除成员函数

不可复制习语是一种 C++ 设计模式，用于防止对象被复制或分配。它通常应用于管理资源（如文件句柄或网络套接字）的类，在这些类中，复制对象可能会导致资源泄漏或重复删除等问题。

要使类不可复制，需要删除复制构造函数和复制赋值操作符。这可以在类的声明中明确地进行，让其他程序员清楚地知道不允许复制。
```c++
class NonCopyable {
public:
  NonCopyable() = default;
  ~NonCopyable() = default;

  // Delete the copy constructor
  NonCopyable(const NonCopyable&) = delete;

  // Delete the copy assignment operator
  NonCopyable& operator=(const NonCopyable&) = delete;
};
```

5. Erase-remove

Erase-remove习语是一种常见的 C++ 技术，用于高效地从容器中移除元素，特别是从 std::vector 、 std::list 和 std::deque 等标准序列容器中移除元素。它利用了标准库算法 std::remove （或 std::remove_if ）和成员函数 erase() 。这个习语包括两个步骤：
- `std::remove` （或 `std::remove_if` ）将要移除的元素移到容器的末尾，并返回一个指向第一个要移除元素的迭代器。
- `container.erase()` 使用上一步获得的迭代器从容器中删除元素。

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> numbers = {1, 3, 2, 4, 3, 5, 3};
    
    // Remove all occurrences of 3 from the vector.
    numbers.erase(std::remove(numbers.begin(), numbers.end(), 3), numbers.end());

    for (int number : numbers) {
        std::cout << number << " ";
    }

    return 0;
}
```

7. Copy-Write Idiom

复制-写入习语，有时也称为 "写入时复制"（CoW）或 "懒复制 "，是编程中用于最大限度减少复制大型对象开销的一种技术。它通过使用对象的共享引用，只在需要修改数据时才复制数据，从而有助于减少实际复制操作的次数。

```c++
#include <iostream>
#include <memory>

class MyString {
public:
    MyString(const std::string &str) : data(std::make_shared<std::string>(str)) {}

    // Use the same shared data for copying.
    MyString(const MyString &other) : data(other.data) { 
        std::cout << "Copied using the Copy-Write idiom." << std::endl;
    }

    // Make a copy only if we want to modify the data.
    void write(const std::string &str) {
        // Check if there's more than one reference.
        if(!data.unique()) {
            data = std::make_shared<std::string>(*data);
            std::cout << "Copy is actually made for writing." << std::endl;
        }
        *data = str;
    }

private:
    std::shared_ptr<std::string> data;
};

int main() {
    MyString str1("Hello");
    MyString str2 = str1; // No copy operation, just shared references.

    str1.write("Hello, World!"); // This is where the actual duplication happens.
    return 0;
}
```
在本例中，我们有一个 MyString 类，它模拟了复制-写入成语。当创建一个 MyString 对象时，它会构造一个指向字符串的 shared_ptr 。当复制一个 MyString 对象时，它不会执行任何实际的复制操作，而只是增加共享对象的引用计数。最后，当调用 write 函数时，它会检查数据是否有多个引用，如果有，它就会实际创建一个新副本并更新引用。这样，就可以避免不必要的拷贝，直到真正需要修改时才进行拷贝。