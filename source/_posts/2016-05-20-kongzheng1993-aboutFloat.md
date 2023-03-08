---
layout: post
title: "关于float"
date: 2016-06-10
excerpt: "为什么面试题里float f=3.4是错的？"
tags: [float]
comments: true
---




### Java类型转换 


Java中不同类型之间的变量赋值时，需要先进行类型转换，才能进行赋值。Java类型转换分为自动转换和强制转换两种。 
基本类型间的自动类型转换需要满足以下条件: 

(1).转换双方的类型必须兼容，例如int和long类型就是兼容的，而int和boolean就是不兼容的。 

(2).只能是"窄类型"向"宽类型"转换,也就是目标类型的数据表示范围要比源类型的数据表示范围要大。 




### 数值常量默认类型 
  
1.Java中整型常量数值的默认类型是int类型，如果需要声明long类型的常量 ，需要在数值加上'l'或者'L'. 
  例如:int i = 3; 
       long l = 3L; 
  
2.Java中的浮点型常量数值默认是double类型，如果要声明一个数值为float型，则需要在数值后面加上'f'或者'F'. 
  例如:double d = 3.14; 
       float f = 3.14f; 
   
### float f = 3.4;语句是错误的
3.4数值常量默认情况下是double类型，如果赋值给f,那么将由double转换成float类型，由前面的知识可以知道是不能自动类型转换的，所以可以将float f = 3.4修改成: 

(1)float f = 3.4f; 
(2)float f = (float)3.4; 



