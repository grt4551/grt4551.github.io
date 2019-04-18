---
layout: post
title: "java 算法系列 - 笛卡尔积算法"
date: 2017-8-30 11:02:56
tags : 
    - java
    - 算法 
categories: java算法 
---
{:toc}
## STEP 1 : 什么是笛卡尔积

来自[维基百科-笛卡儿积](https://zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E7%A7%AF)的解释:
>在数学中，两个集合X和Y的笛卡儿积（Cartesian product），又称直积，在集合论中表示为X × Y，是所有可能的有序对组成的集合，其中有序对的第一个对象是X的成员，第二个对象是Y的成员。  
>$${\displaystyle X\times Y=\left\{\left(x,y\right)\mid x\in X\land y\in Y\right\}}$$

来自[百度百科-笛卡儿积](https://baike.baidu.com/item/%E7%AC%9B%E5%8D%A1%E5%B0%94%E4%B9%98%E7%A7%AF/6323173?fromtitle=%E7%AC%9B%E5%8D%A1%E5%B0%94%E7%A7%AF&fromid=1434391)的解释: 
>笛卡尔乘积是指在数学中，两个集合X和Y的笛卡尓积（Cartesian product），又称直积，表示为X × Y，第一个对象是X的成员而第二个对象是Y的所有可能有序对的其中一个成员。

<!--more-->

举个比较常见的例子:

>如果集合X是13个元素的点数集合{ A, K, Q, J, 10, 9, 8, 7, 6, 5, 4, 3, 2 }，而集合Y是4个元素的花色集合{♠, ♥, ♦, ♣}，则这两个集合的笛卡儿积是有52个元素的标准扑克牌的集合{ (A, ♠), (K, ♠), ..., (2, ♠), (A, ♥), ..., (3, ♣), (2, ♣) }。

## STEP 2 : java 实现笛卡尔积算法

实际上网络上有很多的笛卡尔积算法的实现,在此我就取其中一种方法进行演示:

>（1）循环内，每次只有一列向下移一个单元格，就是CounterIndex指向的那列。  
>（2）如果该列到尾部了，则这列index重置为0，而CounterIndex则指向前一列，相当于进位，把前列的index加一。  
>（3）最后，由生成的行数来控制退出循环。 

{% highlight java %}
public class Test {
    public static void main(String[] args) {
        String[] x = {"A", "K", "Q", "J", "10", "9", "8", "7", "6", "5", "4", "3", "2"};
        String[] y = {"♠", "♥", "♦", "♣"};

		String[][] z = new String[y.length][x.length];
        for (int i = 0; i < y.length; i++) {
            z[i] = x;
        }
        String[][] temp = cartesianProduct(z);
        for (int i = 0; i < temp.length; i++) {
            System.out.println(Arrays.toString(temp[i]));
        }
        System.out.println(temp.length);
    }

    private static String[][] cartesianProduct(String[][] args) {
        int total = 1;
        int counterIndex = args.length - 1;
        int[] counter = new int[args.length];
        for (int i = 0; i < args.length; i++) {
            total *= args[i].length;
            counter[i] = 0;
        }

        String[][] result = new String[total][args.length];
        for (int i = 0; i < total; i++) {
            for (int j = 0; j < args.length; j++) {
                result[i][j] = args[j][counter[j]];
            }
            counterIndex = handle(counter, counterIndex, args);
        }
        return result;
    }

    private static int handle(int[] counter, int counterIndex, String[][] args) {
        counter[counterIndex]++;
        if (counter[counterIndex] >= args[counterIndex].length) {
            counter[counterIndex] = 0;
            counterIndex--;
            if (counterIndex >= 0) {
                handle(counter, counterIndex, args);
            }
            counterIndex = args.length - 1;
        }
        return counterIndex;
    }
}
{% endhighlight %}

## STEP 3 : 笛卡尔积算法解决经典问题 

网络上有这样一个问题:
>1 2 3 4 5 6 7 8 9   
>**这九个按顺序排列的数，要求在它们之间插入若干个 + , -  或者什么都不加**  
>**使其结果正好等于100**

我们可以使用笛卡尔积算法将所有的组合计算出来,然后取其中结果为100的组合,具体实现如下:  

{% highlight java %}
public class Test {
    static ScriptEngine jse = new ScriptEngineManager().getEngineByName("JavaScript");

    public static void main(String[] args) throws InterruptedException {
        String[] x = {"+", "-", ""};
        String[] y = {"1", "2", "3", "4", "5", "6", "7", "8", "9"};

        String[][] z = new String[y.length][x.length];
        for (int i = 0; i < y.length; i++) {
            z[i] = x;
        }

        //计算结果
        String[][] result = cartesianProduct(z);
        for (int i = 0; i < result.length; i++) {
            StringBuffer sb = new StringBuffer();
            for (int j = 0; j < result[i].length; j++) {
                sb.append(y[j]);
                sb.append(result[i][j]);
            }
            sb.append(y[y.length - 1]);

            try {
				//利用js脚本引擎直接转换字符串为计算表达式,从而获得计算结果
                Object obj = jse.eval(sb.toString());
                if (Integer.parseInt(String.valueOf(obj)) == 100){
                    System.out.println(sb.toString() + " = 100");
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    private static String[][] cartesianProduct(String[][] args) {
        int total = 1;
        int counterIndex = args.length - 1;
        int[] counter = new int[args.length];
        for (int i = 0; i < args.length; i++) {
            total *= args[i].length;
            counter[i] = 0;
        }

        String[][] result = new String[total][args.length];
        for (int i = 0; i < total; i++) {
            for (int j = 0; j < args.length; j++) {
                result[i][j] = args[j][counter[j]];
            }
            counterIndex = handle(counter, counterIndex, args);
        }
        return result;
    }

    private static int handle(int[] counter, int counterIndex, String[][] args) {
        counter[counterIndex]++;
        if (counter[counterIndex] >= args[counterIndex].length) {
            counter[counterIndex] = 0;
            counterIndex--;
            if (counterIndex >= 0) {
                handle(counter, counterIndex, args);
            }
            counterIndex = args.length - 1;
        }
        return counterIndex;
    }
}
{% endhighlight %}

## 结束

>**本文部分文本来源于互联网**  
>
>**感谢以下文章提供的灵感和帮助**  

> - [笛卡尔积算法的Java实现](http://blog.csdn.net/a9529lty/article/details/7711151)