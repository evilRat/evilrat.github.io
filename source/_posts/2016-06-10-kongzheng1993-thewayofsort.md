---
layout: post
title: "几种常见的内部排序"
date: 2016-06-10
excerpt: "介绍几种经典的内部排序"
tags: [sample post, images, test]
comments: true
---

## 概论

排序有内部排序和外部排序，内部排序是数据记录在内存中进行排序，而外部排序是因排序的数据很大，一次不能容纳全部的排序记录，在排序过程中需要访问外存。

我整理的排序就是内部排序。



当数据较多时应该采用时间复杂度为o(nlog2n)的排序方法：快速排序、堆排序、归并排序

快速排序是这几种内部排序中最好的方法，想待排序的关键字是随机分布时，快速排序的平均时间最短。

 

## 直接插入排序

### 思想

将一个记录插入到已排序好的有序表中，从而得到一个新，记录数增1的有序表。即：先将序列的第1个记录看成是一个有序的子序列，然后从第2个记录逐个进行插入，直至整个序列有序为止。

### 要点
设立哨兵，作为临时存储和判断数组边界之用。

如果碰见一个和插入元素相等的，那么插入元素把想插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序就是排好序后的顺序，所以插入排序是稳定的。

### 代码
```
public void insertSort(int[] a){
    int i, j, k;
for (i = 1; i < a.length; i++) {
             //为a[i]在前面的a[0...i-1]有序区间中找一个合适的位置
            for (j = i - 1; j >= 0; j--)//这里判断是j>=0也可以防止数组越界，很巧妙
                 if (a[j] < a[i])
                     break;
             //如找到了一个合适的位置
             if (j != i - 1) {
                 //将比a[i]大的数据向后移
                 int temp = a[i];
                 for (k = i - 1; k > j; k--)
                     a[k + 1] = a[k];
                 //将a[i]放到正确位置上
                 a[k + 1] = temp;
             }
         }
}
```
### 效率

时间复杂度：O（n^2）.

其他的插入排序有二分插入排序，2-路插入排序。

## 简单选择排序

### 基本思想

在要排序的一组数中，选出最小（或者最大）的一个数与第1个位置的数交换；然后在剩下的数当中再找最小（或者最大）的与第2个位置的数交换，依次类推，直到第n-1个元素（倒数第二个数）和第n个元素（最后一个数）比较为止。

### 操作方法

第一趟，从n 个记录中找出关键码最小的记录与第一个记录交换；

第二趟，从第二个记录开始的n-1 个记录中再选出关键码最小的记录与第二个记录交换；

以此类推.....

第i 趟，则从第i 个记录开始的n-i+1 个记录中选出关键码最小的记录与第i 个记录交换，

直到整个序列按关键码有序。

### 代码
```
public void selectSort(int a[]){
    int index,temp;
    //index保存目前最小的数据的下标
    //找出最小的数据的位置
    for (int i=0;i<a.length ;i++) {
        index=i;//因为每次排完序前面的都是有序的了，前面的肯定比第i个小，所以让index=i，减少不必要的麻烦
        for (int j=i;j<a.length ;j++ ) {
            if (a[j]<a[index]) {
            index=j;
            }        
        }
        System.out.println("第"+i+"次找到的最小值的下标是："+index);
        if(index!=i)
            {
                temp=a[index];//找到无序数列里面的最小值并于当前位置(i)交换
                a[index]=a[i];
                a[i]=temp;
            }
        for (int m=0; m<a.length;m++ ) {
            System.out.print(a[m]+" ");

        }
        System.out.println();
    }
    
}
```
## 简单选择排序的改进 --二元选择排序

简单选择排序，每趟只能确定一个元素排序后的定位，我们可以考虑改进为每趟确定两个元素，也就是最大值和最小值的位置，从而减少循环次数，改进后对n个数据进行排序，最多只需进行[n/2]趟循环。

### 代码
```
//这个算法因为比较的是大小，将min和max都记录下来，交换到当前坐标，但是如果数组中有相同的值，他们也会不论你这是交换还是不交换，都不会改变结果，所以这个方法不适用于有相同数据的数组
void selectSort_double(int a[]){
    int min,max,temp;
    for (int i=0;i<=a.length/2;i++ ) {
        min=i;max=i;
        for (int j=i;j<a.length-i;j++) {
            if(a[j]<a[min]){
                min=j;
                System.out.println("min="+min);
                continue;//如果当前左边的数据小于当前最小值，那么它必定小于最大值，直接进入下一次循环
            }
            if (a[j]>a[max]) {
                max=j;
                System.out.println("max="+max);
            }   
        }
        System.out.println("第"+i+"次找到的最小值的下标是："+min+";第"+i+"次找到的最大值的下标是："+max);
        temp=a[i];a[i]=a[min];a[min]=temp;
        temp=a[a.length-i-1];a[a.length-i-1]=a[max];a[max]=temp;
        for (int w=0;w<a.length ;w++ ) {
            System.out.print(a[w]+" ");
        }
        temp=a[a.length-i-1];a[a.length-i-1]=a[max];a[max]=temp;
        System.out.println();
    }
}
```
## 冒泡排序

### 基本思想

冒泡排序是相继比较交换两个相邻位置的值，每次排序都确定一个值的位置，就像冒泡一样

### 代码

```
public class BubbleSort1{

    public void bubble(int a[]){

        for (int i=0;i<a.length;i++ ) {
            for (int j=0;j<a.length-i-1 ;j++ ) {
                
                if (a[j]>a[j+1]) {
                    int temp=a[j];
                    a[j]=a[j+1];
                    a[j+1]=temp;
                }
                // out(a);

            }
            out(a);
}
    }
    public void out(int a[]){
        for (int i=0;i<a.length ;i++ ) {
            System.out.print(a[i]+" ");
        }
        System.out.println();
    }
    public static void main(String[] args) {
        BubbleSort1 bu=new BubbleSort1();
        int arr[]={2,5,1,3,6,9,7};
        // bu.out(arr);
        bu.bubble(arr);
        // bu.out(arr);
        System.out.println("hello world");
    }


}

```


<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-thewayofsort/" data-title="thewayofsort" data-url="http://kongzheng1993.github.io/kongzheng1993"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"kongzheng1993"};
    (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
    })();
</script>
</html>
