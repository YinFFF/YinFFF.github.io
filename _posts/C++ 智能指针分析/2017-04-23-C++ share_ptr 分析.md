---
layout: post
title: C++ share_ptr分析
date: 2017-04-23
categories: C/C++
tags: 智能指针 C++11
---

* content
{:toc}

## 特性

1. share_ptr 属于智能指针中的一种，本质上是类，其特点为一个对象的控制权可以被多个share_ptr共享，其内存中结构如下：

   <img src="/images/share_ptr.png" width="60%" />

2. share_ptr 可以如下构造：

   ```c++
   int main(int argc, char* argv[])
   {
   	int *mem = new int;
   	std::shared_ptr<int> test1(mem);
   	std::shared_ptr<int> test2 = test1;
     	std::shared_ptr<int> test3(new int);
   }
   ```

3. share_ptr的错误赋值：

   ```c++
   int main(int argc, char* argv[])
   {
   	int *mem = new int;
   	std::shared_ptr<int> test1;
     	test1 = mem;  //无法从指针直接赋值给share_ptr
   }
   ```

4. 通过赋值操作，可以将两个share_ptr的指针指向同一区域，计数区为2:

   ```c++
   int main(int argc, char* argv[])
   {
   	int *mem = new int;
   	std::shared_ptr<int> test1(mem);
   	std::shared_ptr<int> test2 = test1;
   }
   ```

   <img src="/images/share_ptr2.png" width="60%" />

   ​

5. 如果直接将一个动态空间传给给两个share_ptr的构造函数，会导致计数区出现问题：

   ```c++
   int main(int argc, char* argv[])
   {
   	int *mem = new int;
   	std::shared_ptr<int> test1(mem);
   	std::shared_ptr<int> test2(mem);
   }
   ```


<img src="/images/share_ptr3.png" width="60%" />


   产生这种情况的原因很明显，因为计数区在堆中，test2的构造函数无法知道mem对象是否已经被其他share_ptr所指向了。这种情况会导致test1和test2的析构函数对mem进行两次内存释放，程序会出错。

   同理，enable_shared_from_this 类在share_ptr中的使用的缘由，就是为了解决类似的问题。



## enable_shared_from_this

1. 在类的成员函数中有可能会以两种方式使用this指针：
   - 将this作为返回值
   - 在调用其他函数时，将this作为函数实参
2. 在当前类实例是动态申请出来的且由shared_ptr来控制的时候，以上两个步骤就无法使用this指针了，因为既然要将当前类的地址传给其他函数，往往调用的函数形参也是shared_ptr类型，所以此时增加了计数区的值，而this本身因为不是shared_ptr类型的，当其作为实参时会造成前面所说的问题，因此需要有一种机制避免这种情况发生。