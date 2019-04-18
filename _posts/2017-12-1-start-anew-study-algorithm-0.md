---
layout: post
title:  "算法从头学(0) - 算法中常见的概念"
date: 2017-12-1 17:53:38
tags: 
    - java
    - Algorithm
categories: 算法从头学 
---

{:toc}

# 什么是算法
>&emsp; In mathematics and computer science, an algorithm is an unambiguous specification of how to solve a class of problems. Algorithms can perform calculation, data processing and automated reasoning tasks.  
>&emsp;在数学和计算机科学中，算法是如何解决一类问题的明确规范。 算法可以执行计算，数据处理和自动推理任务。 
><p align="right">--- <a href="https://en.wikipedia.org/wiki/Algorithm">维基百科</a></p>

通俗的来讲，人们在给定一个输入条件由一定的**简单计算步骤**过后总能产生正确的输出结果，我们就把这些计算步骤统称为算法。

# 算法复杂度
每一个算法完成计算所需要花费的时间叫做`时间复杂度 (Time Complexity)`,完成计算所需要花费的空间叫做`空间复杂度 (Space Complexity)`。通常来说，时间和空间两个维度是矛盾的，一个算法无法同时兼顾快速运行和低空间消耗。

在现代计算机中，由于存储空间的大幅提升所导致算法的空间复杂度已经被忽略了，所以本篇文章仅针对时间复杂度进行探讨。

<!--more-->

## 复杂度的表示
### 时间复杂度表示
国际通用的算法复杂度的表示是使用的`大O符号` ([维基百科](https://zh.wikipedia.org/wiki/%E5%A4%A7O%E7%AC%A6%E5%8F%B7))表示的。

>一般情况下，算法中基本操作重复执行的次数是问题规模n的某个函数，用$$T(n)$$表示，若有某个辅助函数$$f(n)$$,使得当n趋近于无穷大时，$$T（n)/f(n)$$的极限值为不等于零的常数，则称$$f(n)$$是$$T(n)$$的同数量级函数。记作$$T(n)=Ｏ(f(n))$$,称$$Ｏ(f(n))$$ 为算法的渐进时间复杂度，简称时间复杂度。

例如：
>  $$O(n^2)$$
   
这个公式表示算法的复杂度随着n的增长会以n的平方结果增长。


### 空间复杂度表示
>一个算法所需的存储空间用$$f(n)$$表示。$$S(n)=O(f(n))$$其中n为问题的规模，S(n)表示空间复杂度。

## 时间复杂度的计算
对于一个方法的时间复杂度的计算，需要记住以下几点：

	1. 找到执行次数最多的语句
	2. 计算语句执行次数的数量级
	3. 忽略常数级时间复杂度
	4. 用大O来表示结果  

### 例子1：顺序查找
{% highlight java linenos%}
public void sequenceSearch(char c,char[] chars){
	int n = chars.length;
    for (int i = 0; i < n; i++) {
        if (chars[i] == c){
            System.out.println("找到了，在第" + i);
            return;
        }
    }
    System.out.println("没有找到");
}
{% endhighlight %}
根据上面的规则，我们可以看出这个例子里：

	1. 执行次数最多的是for循环。
	2. 循环体内语句时间复杂度为O(1)。
	3. 随着c的变化，该方法的时间复杂度最大的是O(n),最小的是O(1)。

所以我们可以得出，这个算法的平均时间复杂度为：$$O(n/2)$$

### 例子2.递归算法 获取斐波那契数列中指定位置的数
{% highlight java linenos%}
public static long getFib(long index) {
    if (index <= 1) {
        return index;
    } else {
        return getFib(index - 1) + getFib(index - 2);
    }
}
{% endhighlight %}
这个例子中的获取斐波那契数的方法是使用递归，具体算法复杂度分析如下：

>我们假设T(n)代表getFib()方法的调用次数  
> 当n=0或1时，T(n)=1;  
> 当n=2时，$$T(2)=T(1)+T(0)+1=2+1=2^(n-1)+1=2^n-1$$;  
> 当n=3时，  
> $$T(3)=T(2)+T(1)+1$$  
> &emsp;&emsp;$$=(T(1)+T(0)+1)+T(1)+1$$  
> &emsp;&emsp;$$=2\times T(1)+T(0)+2$$   
> &emsp;&emsp;$$=2\times 1+1+2$$  
> &emsp;&emsp;$$=2^2+1$$  
> &emsp;&emsp;$$=2^(n-1)+1$$  
> &emsp;&emsp;$$=2^n-1$$;  

从上面的规律我们可以得出，$$T(n)=2^n-1$$,根据我们上面所说的忽略常数级时间复杂度，我们可以的到该算法的时间复杂度为：$$O(2^n)$$


### 例子3.通项公式算法 获取斐波那契数列中指定位置的数
{% highlight java linenos%}
private static final double sqrt_5 = Math.sqrt(5);
public static long getFibonacci(long index) {
    return new Double(1.00 / sqrt_5 * (Math.pow((1 + sqrt_5) / 2, index) - Math.pow((1 - sqrt_5) / 2, index))).longValue();
}
{% endhighlight %}
这个例子就非常简单了，所有的方法都是一次性调用，且没有循环的方法，所以它的时间复杂度为：$$O(1)$$

## 总结
从例子2和例子3我们可以看见：为了实现同一个功能，算法的不同会导致时间复杂度的巨大差异。所以，在实际项目中我们需要详细分析自己的需求以寻找最合适的实现算法。

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [Wikipedia - Algorithm](https://en.wikipedia.org/wiki/Algorithm)  
> - [维基百科 - 大O符号](https://zh.wikipedia.org/wiki/%E5%A4%A7O%E7%AC%A6%E5%8F%B7)
> - [（数据结构）十分钟从零搞定时间复杂度（计算算法的时间复杂度）](http://www.jianshu.com/p/f4cca5ce055a)  
> - [算法的时间复杂度](https://www.guokr.com/blog/68001/)  
> - [算法时间复杂度计算](http://www.jianshu.com/p/99bac69fdd97)  
> - [时间复杂度和空间复杂度详解](http://blog.csdn.net/booirror/article/details/7707551/)