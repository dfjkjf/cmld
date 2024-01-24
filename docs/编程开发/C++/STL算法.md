---
sort: 2
---

# STL算法


-   算法主要由头文件`<algorithm>`,`<functional>`,`<numeric>`组成
-   是所有STL头文件中最大的一个，范围涉及到比较、交换、查找、遍历、复制、删除等
-   体积很小，只包括几个在序列上面进行简单数学运算的模板函数
-   定义了一些模板类，用来声明函数对象

## 总结

| 算法 | 参数 | 作用 |
| --- | --- | --- |
| **遍历** |  |  |
| void for_each(iterator beg, iterator end, func()) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`func()`//函数或函数对象 | 遍历容器 |
| void transform(iterator beg1, iterator end1, iterator beg2, func()) | `beg1`//源容器开始迭代器</br>`end1`//源容器结束迭代器</br>`beg2`//目标容器开始迭代器</br>`func`//函数或函数对象 | 搬运容器 |
| **查找** |  |  |
| iterator find(iterator beg, iterator end, value) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`value`//查找的元素 | 按值查找 |
| iterator find_if(iterator beg, iterator end, func()) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`func()`//函数或谓词（返回bool的仿函数） | 条件查找 |
| adjacent_find(iterator beg, iterator end) | `beg`//起始迭代器</br>`end`//结束迭代器 | 查找相邻重复元素 |
| bool binary_search(iterator beg, iterator end, value) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`value`//查找的元素 | 二分查找 |
| int count(iterator beg, iterator end, value) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`value`//查找的元素 | 统计元素个数 |
| int count_if(iterator beg, iterator end, func()) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`func()`//函数或谓词（返回bool的仿函数） | 按条件统计元素个数 |
| **排序** |  |  |
| sort(iterator beg, iterator end, func) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`func()`//函数或者谓词，可填可不填，不填则默认升序排列 | 对容器内元素进行排序 |
| random_shuffle(iterator beg, iterator end) | `beg`//起始迭代器</br>`end`//结束迭代器 | 随机洗牌 |
| merge(iterator beg1, iterator end1, iterator beg2, iterator end2, dest) | `beg1`//容器1开始迭代器</br>`end1`//容器1结束迭代器</br>`beg2`//容器2开始迭代器</br>`end2`//容器2结束迭代器</br>`dest`//目标容器开始迭代器 | 将两个有序容器元素合并，并储存到另一个容器中 |
| reverse(iterator beg, iterator end) | `beg`//起始迭代器</br>`end`//结束迭代器 | 反转指定范围的元素 |
| **拷贝** |  |  |
| copy(iterator beg, iterator end, iterator dest) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`dest`//目标容器开始迭代器 | 容器内指定范围的元素拷贝到另一个容器中 |
| **替换** |  |  |
| replace(iterator beg, iterator end, oldvalue, newvalue) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`oldvalue`//旧元素</br>`newvalue`//新元素 | 将容器内指定范围的旧元素修改为新元素 |
| replace_if(iterator beg, iterator end, func, newvalue) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`func()`//函数或者谓词</br>`oldvalue`//旧元素 | 将容器内满足条件的旧元素修改为新元素 |
| swap(container c1, container c2) | `c1`//容器1</br>`c2`//容器2 | 互换两个的容器元素 |
| **算术** |  |  |
| accumulate(iterator beg, iterator end, value) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`value`//起始值 | 计算容器元素累计总和 |
| fill(iterator beg, iterator end, value) | `beg`//起始迭代器</br>`end`//结束迭代器</br>`value`//填充的值 | 向容器中添加元素 |
| **集合** |  |  |
| set_intersection(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest) | `beg1`//容器1开始迭代器</br>`end1`//容器1结束迭代器</br>`beg2`//容器2开始迭代器</br>`end2`//容器2结束迭代器</br>`dest`//目标容器开始迭代器 | 求两个容器的交集 |
| set_union(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest) | `beg1`//容器1开始迭代器</br>`end1`//容器1结束迭代器</br>`beg2`//容器2开始迭代器</br>`end2`//容器2结束迭代器</br>`dest`//目标容器开始迭代器 | 求两个容器的并集 |
| set_difference(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest) | `beg1`//容器1开始迭代器</br>`end1`//容器1结束迭代器</br>`beg2`//容器2开始迭代器</br>`end2`//容器2结束迭代器</br>`dest`//目标容器开始迭代器 | 求两个容器的差集 |



## 常用遍历算法

### for_each

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//普通函数
void myPrint(int val)
{
    cout << val << " ";
}
//仿函数
class myPrint2
{
public:
    void operator()(int val)
    {
        cout << val << " ";
    }
};
int main()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    for_each(v.begin(), v.end(), myPrint);
    cout << endl;
    for_each(v.begin(), v.end(), myPrint2());
    return 0;
}
```

### transform

搬运的目标容器必须提前开辟空间

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
class Transform
{
public:
    int operator()(int val)
    {
        return val;
    }
};
int main()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    vector<int>v2;
    v2.resize(v.size());
    transform(v.begin(), v.end(), v2.begin(), Transform());
    for (vector<int>::iterator it = v2.begin(); it != v2.end(); it++)
    {
        cout << *it << " ";
    }
    return 0;
}
```

## 常用查找算法

### find

返回一个迭代器，如果没有找到，返回end()  
查找自定义数据类型需要重载==运算符，否则底层不知道如何对比

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//查找内置数据类型
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    //返回一个迭代器，如果没有找到，返回end()
    vector<int>::iterator it = find(v.begin(), v.end(), 5);
    if (it == v.end())
        cout << "没找到" << endl;
    else
        cout << "找到了" << *it << endl;
}
//查找自定义数据类型
class Person
{
public:
    Person(string name,int age)
    {
        this->m_age = age;
        this->m_name = name;
    }
    //重载==运算符，让find知道如何对比Person类型数据
    bool operator==(const Person& p)
    {
        if (p.m_age == this->m_age && p.m_name == this->m_name)
            return true;
        else
            return false;
    }
    string m_name;
    int m_age;
};
void test02()
{
    //准备数据
    Person p1("A", 1);
    Person p2("B", 2);
    Person p3("C", 3);
    Person p4("D", 4);
    Person p5("E", 5);
    //放入容器中
    vector<Person>p;
    p.push_back(p1);
    p.push_back(p2);
    p.push_back(p3);
    p.push_back(p4);
    p.push_back(p5);
    //查找
    Person p6("A", 1);
    vector<Person>::iterator it = find(p.begin(), p.end(), p6);
    //输出，验证结果
    if (it == p.end())
        cout << "没找到" << endl;
    else
        cout << "找到了" << it->m_name << it->m_age << endl;
}
int main()
{
    test01();
    test02();
    return 0;
}
```

### find_if

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//查找内置数据类型
class GreaterFive
{
public:
    bool operator()(int v)
    {
        return v > 5;
    }
};
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    vector<int>::iterator it = find_if(v.begin(), v.end(), GreaterFive());
    if (it == v.end())
        cout << "没找到" << endl;
    else
        cout << "找到了" << *it << endl;
}
//查找自定义数据类型
class Person
{
public:
    Person(string name, int age)
    {
        this->m_age = age;
        this->m_name = name;
    }
    string m_name;
    int m_age;
};
class GreaterThree
{
public:
    bool operator()(Person& p)
    {
        return p.m_age > 3;
    }
};
void test02()
{
    //准备数据
    Person p1("A", 1);
    Person p2("B", 2);
    Person p3("C", 3);
    Person p4("D", 4);
    Person p5("E", 5);
    //放入容器中
    vector<Person>p;
    p.push_back(p1);
    p.push_back(p2);
    p.push_back(p3);
    p.push_back(p4);
    p.push_back(p5);
    //查找
    vector<Person>::iterator it = find_if(p.begin(), p.end(), GreaterThree());
    //输出，验证结果
    if (it == p.end())
        cout << "没找到" << endl;
    else
        cout << "找到了" << it->m_name << it->m_age << endl;
}
int main()
{
    test01();
    test02();
    return 0;
}
```

### adjacent_find
如果查到，返回相邻重复元素的第一个位置的迭代器
```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    v.push_back(5);
    vector<int>::iterator it=adjacent_find(v.begin(), v.end());
    if (it == v.end())
        cout << "未找到相邻重复元素" << endl;
    else
        cout << "找到了相邻重复元素" << *it << endl;
}
int main()
{
    test01();
    return 0;
}
```

### binary_search

- 注意：在无序列表中不可用，如果是无序序列，结果未知

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    cout << binary_search(v.begin(), v.end(), 5) << endl;
}
int main()
{
    test01();
    return 0;
}
```

### count

对于统计自定义数据类型，需要重载==运算符

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//统计内置数据类型
void test01()
{
    vector<int>v;
    v.push_back(10);
    v.push_back(20);
    v.push_back(10);
    cout << count(v.begin(), v.end(), 10) << endl;
}
//统计自定义数据类型
class Person
{
public:
    Person(string name, int age)
    {
        this->m_name = name;
        this->m_age = age;
    }
    //需要重载==运算符
    //底层要求加const
    bool operator==(const Person& p)
    {
        if (this->m_name == p.m_name)
            return true;
    }
    string m_name;
    int m_age;
};
void test02()
{
    Person p1("A", 1);
    Person p2("B", 2);
    Person p3("A", 3);
    Person p4("A", 4);
    vector<Person>p;
    p.push_back(p1);
    p.push_back(p2);
    p.push_back(p3);
    cout << "与p4同名的元素个数" << count(p.begin(), p.end(), p4) << endl;
}
int main()
{
    test01();
    test02();
    return 0;
}
```

### count_if

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//统计内置数据类型
class GreaterFive
{
public:
    bool operator()(int val)
    {
        return val > 5;
    }
};

void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    cout << count_if(v.begin(), v.end(), GreaterFive()) << endl;
}
//统计自定义数据类型
class Person
{
public:
    Person(string name, int age)
    {
        this->m_name = name;
        this->m_age = age;
    }
    string m_name;
    int m_age;
};
class AgeGreaterTwo
{
public:
    bool operator()(const Person& p)
    {
        return p.m_age > 2;
    }
};
void test02()
{
    Person p1("A", 1);
    Person p2("B", 2);
    Person p3("A", 3);
    Person p4("A", 4);
    vector<Person>p;
    p.push_back(p1);
    p.push_back(p2);
    p.push_back(p3);
    p.push_back(p4);
    //统计年龄大于2的人数
    cout << count_if(p.begin(), p.end(), AgeGreaterTwo()) << endl;
}
int main()
{
    test01();
    test02();
    return 0;
}
```

## 常用排序算法

### sort

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    //默认升序
    sort(v.begin(), v.end());
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++)
    {
        cout << *it << " " ;
    }
    cout << endl;
    //使用内建函数对象实现降序排列
    sort(v.begin(), v.end(), greater<int>());
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++)
    {
        cout << *it << " ";
    }
}
int main()
{
    test01();
    return 0;
}
```

### random_shuffle随机洗牌

`srand((unsigned int)time(NULL));`可以设置系统时间为随机数种子

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    random_shuffle(v.begin(), v.end());
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++)
    {
        cout << *it << " " ;
    }
    cout << endl;
}
int main()
{
    test01();
    return 0;
}
```

### merge

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void Print(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    vector<int>v2;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
        v2.push_back(i + 1);
    }
    //目标容器
    vector<int>v3;
    //目标容器需要提前开辟空间
    v3.resize(v1.size() + v2.size());
    merge(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
    //合并后仍然是有序容器
    for_each(v3.begin(), v3.end(), Print);
}
int main()
{
    test01();
    return 0;
}
```

### reverse

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void Print(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
    }
    for_each(v1.begin(), v1.end(), Print);
    cout << endl;
    //反转
    reverse(v1.begin(), v1.end());
    for_each(v1.begin(), v1.end(), Print);
}
int main()
{
    test01();
    return 0;
}
```

## 常用拷贝算法

### copy

用到的比较少，直接用赋值操作更简单

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
void Print(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
    }
    vector<int>v2;
    //v2要提前开辟空间
    v2.resize(v1.size());
    copy(v1.begin(), v1.end(), v2.begin());
    for_each(v2.begin(), v2.end(), Print);
}
int main()
{
    test01();
    return 0;
}
```

## 常用替换算法

### replace

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
class Print
{
public:
    void operator()(int val)
    {
        cout << val << " ";
    }
};
void test01()
{
    vector<int>v1;
    v1.push_back(1);
    v1.push_back(2);
    v1.push_back(1);
    v1.push_back(3);
    for_each(v1.begin(), v1.end(), Print());
    cout << endl;
    //替换
    replace(v1.begin(), v1.end(), 1, 2);
    for_each(v1.begin(), v1.end(), Print());
}
int main()
{
    test01();
    return 0;
}
```

### replace_if

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
class Print
{
public:
    void operator()(int val)
    {
        cout << val << " ";
    }
};
class GreaterFive
{
public:
    bool operator()(const int& val)
    {
        return val > 5;
    }
};
void test01()
{
    vector<int>v;
    for (int i = 0; i < 10; i++)
    {
        v.push_back(i);
    }
    replace_if(v.begin(), v.end(), GreaterFive(), 0);
    for_each(v.begin(), v.end(), Print());
}
int main()
{
    test01();
    return 0;
}
```

### swap

注意必须是同种容器，大小相同

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
class Print
{
public:
    void operator()(int val)
    {
        cout << val << " ";
    }
};
class GreaterFive
{
public:
    bool operator()(const int& val)
    {
        return val > 5;
    }
};
void test01()
{
    vector<int>v1;
    vector<int>v2;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
        v2.push_back(i + 2);
    }
    //交换前
    for_each(v1.begin(), v1.end(), Print());
    for_each(v2.begin(), v2.end(), Print());
    cout << endl;
    //交换后
    swap(v1, v2);
    for_each(v1.begin(), v1.end(), Print());
    for_each(v2.begin(), v2.end(), Print());
}
int main()
{
    test01();
    return 0;
}
```

## 常用算术生成算法

算术生成算法属于小型算法，使用时包含的头文件为

### accumulate

```cpp
#include<iostream>
#include<vector>
#include<numeric>
using namespace std;

void test01()
{
    vector<int>v1;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
    }
    int total = accumulate(v1.begin(), v1.end(), 0);
    cout << total << endl;
}
int main()
{
    test01();
    return 0;
}
```

### fill

```cpp
#include<iostream>
#include<vector>
#include<numeric>
using namespace std;

void test01()
{
    vector<int>v1;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
    }
    for (vector<int>::iterator it = v1.begin(); it != v1.end(); it++)
    {
        cout << *it << " ";
    }
    cout << endl;
    fill(v1.begin(), v1.end(), 0);
    for (vector<int>::iterator it = v1.begin(); it != v1.end(); it++)
    {
        cout << *it << " ";
    }
}
int main()
{
    test01();
    return 0;
}
```

## 常用集合容器

### set_intersection

注意事项：

-   返回值为迭代器，指向交集**最后一个元素的下一个位置**
-   求交集的两个集合必须为有序数列
-   目标容器开辟空间需要从两个容器中**取小值**

交集就是两个容器重复的元素

```cpp
#include<iostream>
#include<vector>
#include<numeric>
#include<algorithm>
using namespace std;
void myPrint(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    vector<int>v2;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
        v2.push_back(i + 2);
    }
    for_each(v1.begin(), v1.end(), myPrint);
    cout << endl;
    for_each(v2.begin(), v2.end(), myPrint);
    cout << endl;
    //目标容器需要提前开辟空间，最特殊的情况，大容器包含小容器
    vector<int>v3;
    v3.resize(min(v1.size(), v2.size()));
    //取交集
    vector<int>::iterator itEnd = set_intersection(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
    for_each(v3.begin(), itEnd, myPrint);
    cout << endl;
    //如果不用返回的itEnd，会把0也给打印出来
    for_each(v3.begin(), v3.end(), myPrint);
}
int main()
{
    test01();
    return 0;
}
```

### set_union

注意事项：

-   返回值为迭代器，指向并集**最后一个元素的下一个位置**
-   求并集的两个集合必须为有序数列
-   目标容器开辟空间需要**两个容器相加**

```cpp
#include<iostream>
#include<vector>
#include<numeric>
#include<algorithm>
using namespace std;
void myPrint(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    vector<int>v2;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
        v2.push_back(i + 2);
    }
    for_each(v1.begin(), v1.end(), myPrint);
    cout << endl;
    for_each(v2.begin(), v2.end(), myPrint);
    cout << endl;
    //目标容器需要提前开辟空间，最特殊的情况，两个容器没有交集
    vector<int>v3;
    v3.resize(v1.size() + v2.size());
    //取并集
    vector<int>::iterator itEnd = set_union(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
    for_each(v3.begin(), itEnd, myPrint);
    cout << endl;
    //如果不用返回的itEnd，会把0也给打印出来
    for_each(v3.begin(), v3.end(), myPrint);
}
int main()
{
    test01();
    return 0;
}
```

### set_difference

注意事项：

-   返回值为迭代器，指向并集**最后一个元素的下一个位置**
-   求并集的两个集合必须为有序数列
-   目标容器开辟空间需要从两个容器中**取大值**

```cpp
#include<iostream>
#include<vector>
#include<numeric>
#include<algorithm>
using namespace std;
void myPrint(int val)
{
    cout << val << " ";
}
void test01()
{
    vector<int>v1;
    vector<int>v2;
    for (int i = 0; i < 10; i++)
    {
        v1.push_back(i);
        v2.push_back(i + 2);
    }
    for_each(v1.begin(), v1.end(), myPrint);
    cout << endl;
    for_each(v2.begin(), v2.end(), myPrint);
    cout << endl;
    //目标容器需要提前开辟空间，最特殊的情况，大容器和小容器没有交集
    //取两个容器中大的size作为目标容器开辟空间
    vector<int>v3;
    v3.resize(max(v1.size(),v2.size()));
    //取并集
    vector<int>::iterator itEnd = set_difference(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
    for_each(v3.begin(), itEnd, myPrint);
    cout << endl;
    //如果不用返回的itEnd，会把0也给打印出来
    for_each(v3.begin(), v3.end(), myPrint);
}
int main()
{
    test01();
    return 0;
}

```
