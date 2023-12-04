---
title: 【编程学习】C++中类的定义与使用
typora-root-url: 【编程学习】C++中类的定义与使用
description: 初学C++语言，分享C++中类的定义与使用。
cover: /img/blog_img/10.png
tags:
  - C++
  - 类的构建与访问
categories:
  - 编程学习
  - CPP
  - 基础知识
abbrlink: fb4c9113
date: 2023-08-15 19:14:23
---



# 类

直接上代码，构建一个狗狗类：

```c++
#include <iostream>

using namespace std;

class Dog 
{
    public:
        double dog_weight;
        int dog_age;
        string dog_name;

        Dog(string name);
        void run(void);
        void set(double weight, int age);
    private:
        double something;
};

//类的构造函数
Dog::Dog(string name)
{
    cout << "Dog is being created" << endl;
    dog_name = name;
}
/*
简化方案：
Dog::Dog(string name): dog_name(name)
{
    cout << "Dog is being created" << endl;
}
*/

//类的解析函数
Dog::~Dog(void)
{
    cout << "Dog is being deleted" << endl;
}

// 构建类方法的具体实现
void Dog::set(double weight, int age)
{
    dog_weight = weight;
    dog_age = age;
}

void Dog::run(void) 
{
    cout << "The dog " + dog_name + " is running!" << endl;
}

// 主函数
int main() 
{
    Dog dog1("Mark");
    dog1.set(5.6, 3);
    cout << "The name of dog1 is " << dog1.dog_name << endl;
    cout << "The weight of dog1 is " << dog1.dog_weight << endl;
    cout << "The age of dog1 is " << dog1.dog_age << endl;
    dog1.run();

    return 0;
}
```

其实也不用过多的讲解，触类旁通。
