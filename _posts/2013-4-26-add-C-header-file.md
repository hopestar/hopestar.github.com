---
layout: default
title: C语言编写自己的头文件和多文件编译
---
在C语言中经常会需要用到自己写的函数，为了方便，我们会把它写成自己的头文件，到时候引用就可以了。
一般都是在XXX.h 中声明你自己定义的函数，然后XXX.c中实现。
举个栗子：
>
  /*----wbc.h----*/   
     #ifndef   _WBC_H       
     #define   _WBC_H 
     void fuck();     
     #endif   
     /*----wbc.c-----*/  
  #include"wbc.h"   
  void fuck()   
  {   
      printf("fuck!");  
  }   
   
  /*---bbc.c----*/    
  #include"wbc.h" 
  main(){     
      fuck();     
  }   
> 
-----------------------------------------
注意点的就是一般在.h中都是声明，尽量少#include其它头文件，免得重复包含了。
还有一点容易忽略的就是一般我们都是会在当前目录编译头文件，所以include的时候记得用“”而不是""。（对于大文件我就不清楚了，以后了解，还有各种头文件的嵌套包含这些恶心的事我还没去了解）
其中.h的编译的时候都好像没有用到，我也不知道~╮(╯▽╰)╭
 
编译的时候：
>
  gcc -c bbc.c //生成bbc.o  
  gcc -c wbc.c //生成wbc.o  
   
  gcc -o bbc bbc.o wbc.o  
 >  
 
和可以用Makefile  

>
  bbc : bbc.o wbc.o 
      gcc -o bbc bbc.o wbc.o #记住第二行有tab哟~  
  bbc.o : bbc.c 
  wbc.o : wbc.h 
>                
