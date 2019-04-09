---
layout:     post
title:      C++ vector&lt;bool&gt;
subtitle:   
date:       2019-4-8
author:     XP
header-img: img/problems.jpg
catalog: true
tags:  
    - C++  
---

## C++ vector&lt;bool&gt; ##
先看一个题目，选出正确的选项：    

```cpp
#include <vector>  

bool foo()  
{  
    std::vector<bool> v{true, false};  
    auto& x=v[1];  
    x = !x;  
    return v[1];  
}  

a. foo() does not compile  
b. foo() returns false  
c. foo() returns true  

```

### 1, vector
首先看一下**vector**：  
**vector** 的数据安排以及操作方式，与**array** 非常相似，两者的唯一差别在于空间的运用的灵活性。**array** 是静态空间，一旦配置了就不能改变；如果需要更改空间的大小，需要客户自己申请新空间、拷贝元素以及释放原来的空间。**vector** 是动态空间，随着元素的加入，它的内部机制会自动扩充空间以容纳新元素。  
**vector** 支持随机存取，为支持数组下标[]方式存取元素，需要对[]进行运算符重载, 我们可以看一下[SGI STL vector的源码](https://github.com/karottc/sgi-stl/blob/master/stl_vector.h),下面只列出跟题目相关的部分：  

```cpp  
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >  
class vector : protected _Vector_base<_Tp, _Alloc>  
{  
public:  
  // vector 的嵌套类型定义  
  typedef _Tp value_type;  
  typedef value_type& reference;  

  reference operator[](size_type __n) { return *(begin() + __n); }  

}；  

```  

可以看到，*operator[]\()* 返回一个reference，即value_type&；直观地来说，像vector&lt;int&gt vInt, vInt[0]的类型就是int&。把题目中改成这样：  
  
```cpp
#include <vector>  

int foo()  
{  
    std::vector<int> vInt{1, 2};  
    auto& x=vInt[1];  // it is auto&  NOT auto
    x = 10;  
    return vInt[1];  
}  

```  

上面的代码输出 10 

> 1.当不声明为引用时，auto的初始化表达式即使是引用，编译器也不会将其推导为对应的类型的引用，所以上面代码是auto& x=vInt[1]; auto类型推导，请参考[条款二：理解auto类型推导](https://www.kancloud.cn/kangdandan/book/169971)  
> 
> 2.关于如何在C++中打印变量的类型，我们可以参考[这里](https://stackoverflow.com/questions/81870/is-it-possible-to-print-a-variables-type-in-standard-c/20170989#20170989)  

### 2，vector&lt;bool&gt;
再来看一下**vector&lt;bool&gt;**：   
首先来打印下下面类型的名字：  

```cpp
#include <iostream>  
#include <vector>  
#include <type_traits>  
#include <typeinfo>  
#ifndef _MSC_VER  
#include <cxxabi.h>  
#endif  
#include <memory>  
#include <string>  
#include <cstdlib>  

using namespace std;

template <class T>  
std::string  
type_name()  
{  
    typedef typename std::remove_reference<T>::type TR;  
    std::unique_ptr<char, void(*)(void*)> own  
        (  
#ifndef _MSC_VER  
         abi::__cxa_demangle(typeid(TR).name(), nullptr,nullptr, nullptr),  
#else  
         nullptr,  
#endif  
         std::free  
        );  
    std::string r = own != nullptr ? own.get() : typeid(TR).name();  
    if (std::is_const<TR>::value)  
        r += " const";  
    if (std::is_volatile<TR>::value)  
        r += " volatile";  
    if (std::is_lvalue_reference<T>::value)  
        r += "&";  
    else if (std::is_rvalue_reference<T>::value)  
        r += "&&";  
    return r;  
}  

int main()  
{  
    vector<int> vInt{1,2};  
    vector<bool> vBool{true,false};  

    //decltype(vInt) is std::vector<int, std::allocator<int> >  
    cout << "decltype(vInt) is " << type_name<decltype(vInt)>() << '\n';  

    //decltype(vInt[0]) is int&   
    cout << "decltype(vInt[0]) is " << type_name<decltype(vInt[0])>() << '\n';   

    //decltype(vBool) is std::vector<bool, std::allocator<bool> >  
    cout << "decltype(vBool) is " << type_name<decltype(vBool)>() << '\n';  

    //decltype(vBool[0]) is std::_Bit_reference  
    cout << "decltype(vBool[0]) is " << type_name<decltype(vBool[0])>() << '\n';   
}  

```

可以看出vBool[0]是 std::_Bit_reference，而不是bool&，这是因为vector&lt;bool&gt;并非通常意义的vector容器，而是[std::vector 对类型 bool 提高空间利用率的特化](https://zh.cppreference.com/w/cpp/container/vector_bool)，bool类型占用一个字节，但是vector&lt;bool&gt;将它优化成一个bit，即每个元素只占用一个bit。我们无法针对一个bit进行取地址或引用的操作，我们可以看看[SGI STL vector&lt;bool&gt;的源码](https://github.com/karottc/sgi-stl/blob/master/stl_bvector.h):  

```cpp
struct _Bit_reference {...};  

class __BVECTOR : public __BVECTOR_BASE   
{  
public:  
  typedef bool value_type;  
  typedef _Bit_reference reference;  

  reference operator[](size_type __n)  
    { return *(begin() + difference_type(__n)); }  
};  

```  

使用operator[]对于普通容器来说，返回的是对应元素的引用；但是对于vector&lt;bool&gt;来说，返回的是_Bit_reference这个内部代理类的临时变量--右值，而不是真正的reference。更详细的说明可以看[这里](https://www.boost.org/sgi/stl/bit_vector.html)  

回到一开始的题目， *auto& x = v[1];* v[1]返回一个右值，无法对一个右值取引用，所以编译报错：<font color="#dd0000">error: invalid initialization of non-const reference of type ‘std::_Bit_reference&’ from an r                                value of type ‘std::vector<bool>::reference {aka std::_Bit_reference}’ </font>  
去掉&，才可以正常编译，并且修改v[1]的值。

### 3，如何使用vector&lt;bool&gt;

避免使用vector&lt;bool&gt;，参考[Item 18. Avoid using vectorr&lt;bool&gt;](http://www.uml.org.cn/c++/pdf/EffectiveSTL.pdf), 使用deque&lt;bool&gt; 或 bitset代替