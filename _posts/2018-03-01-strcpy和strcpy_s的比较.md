---
layout:     post                    # 使用的布局（不需要改）
title:      strcpy和strcpy_s的比较              # 标题 
subtitle:   字符串 #副标题
date:       2018-03-01              # 时间
author:     BY                      # 作者
header-img: img/wkj.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 学习
---

## Hey
##strcpy_s和strcpy()函数的功能几乎是一样的。strcpy函数，就象gets函数一样，它没有方法来保证有效的缓冲区尺寸，所以它只能假定缓冲足够大来容纳要拷贝的字符串。在程序运行时，这将导致不可预料的行为。用strcpy_s就可以避免这些不可预料的行为。
这个函数用两个参数、三个参数都可以，只要可以保证缓冲区大小。

三个参数时：

errno_t strcpy_s(char *strDestination, size_t numberOfElements, const char *strSource );

两个参数时：

errno_t strcpy_s( char (&strDestination)[size], const char *strSource ); // C++ only

例子：

#include<iostream>

#include<cstring>

using namespace std;
 
void Test(void)

{

char *str1=NULL;

str1=new char[20];

char str[7];

strcpy_s(str1,20,"hello world");//三个参数

strcpy_s(str,"hello");//两个参数但如果：char *str=new char[7];会出错：提示不支持两个参数

cout<<"strlen(str1):"<<strlen(str1)<<endl<<"strlen(str):"<<strlen(str)<<endl;

printf(str1);

printf("\n");

cout<<str<<endl;

}
 
int main()

{

Test();

return 0;

}

#include<iostream>

#include<string.h>

using namespace std; 

void Test(void){char *str1=NULL;str1=new char[20];

char str[7];strcpy_s(str1,20,"hello world");//三个参数

strcpy_s(str,"hello");//两个参数但如果：char *str=new char[7];会出错：提示不支持两个参数

cout<<"strlen(str1):"<<strlen(str1)<<endl<<"strlen(str):"<<strlen(str)<<endl;

printf(str1);

printf("\n");

cout<<str<<endl;

} 

int main()

{

Test();

return 0;

}

输出为：

strlen(str1): 11        //另外要注意：strlen(str1)是计算字符串的长度，不包括字符串末尾的“\0”!!!

strlen(str): 5

hello world

hello
