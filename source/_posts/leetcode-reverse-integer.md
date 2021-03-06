layout: article
title: "LeetCode刷题心得——ReverseInteger"
date: 2015-03-20 19:30:14
tags: [LeetCode, C/C++, math]
categories: 编程
---
###Reverse Integer
>Total Accepted: 61132 Total Submissions: 219023
>Reverse digits of an integer.

>Example1: x = 123, return 321
>Example2: x = -123, return -321

&emsp;&emsp;初看这道题觉得挺简单的，只要把整数的每一位都计算出来，然后转换成字符串，再把字符串倒序，然后把新的字符串转回int类型就可以了，转化过程就是不断的当前sum乘10加当前最低位。照着这个想法写了第一版的程序，代码如下：
<!--more-->
```C++	
	class Solution {
	public:
	    int reverse(int x) {
	        int sign = 1;
	        if (x < 0) sign = -1;
	        x = x / sign;
	        string result;
	        while (x != 0) {
	            result.append(1, x % 10 + '0');
	            x /= 10;
	        }
	        int now = 0;
	        for (int i = 0; i < result.size(); i++) {
	            now = now * 10 + (result[i] - '0');
	        }
	        return now * sign;
	    }
	};
```
&emsp;&emsp;结果提交上去发现返回Runtime Error，也就是说程序中间越界了，这时发现了题面下方有个spoiler（剧透）按钮，点开发现里面是一些提示，看完之后才发现原来刚才少考虑了好多东西，例如，当原整数是以若干个零结尾怎么处理？例如100,12000等等，其次如果原整数的逆序数超过了int能够表示的范围（32位，-2147483648~-2147483647），例如1000000003怎么办？这时才发现自己考虑问题有点简单，一些重要的细节都没有考虑清楚就开始写是不正确的，以后这一点需要注意。
&emsp;&emsp;发现了问题之后就开始考虑如何处理，首先关于若干个零的问题无需特别处理，因为在程序中前面的0都会在累加过程中都会被忽略，那么就是最主要的整数越界问题，首先想到的方法是在累加过程中会产生越界，所以用整数上限减当前最低位然后除以10再和当前sum比较就能判断数字是否越界，这个思想是将会产生越界现象的加法乘法运算转换成不会产生越界的减法除法运算，以便于判断是否会产生越界。
&emsp;&emsp;第二个想法是发现int是有符号整型，能表示的范围是-2^31~2^31-1，那么只需要将这个int放到范围更广的数据类型中就可以解决溢出的问题，我想到的办法是用无符号整型unsigned int，因为这个类型的表示范围是0~2^32-1，而有符号整型的负数可以先变换成正数然后再计算就可以，这样得到的代码如下：
```C++
	class Solution {
	public:
	    int reverse(int x) {
	        int sign = 1;
	        if (x < 0) sign = -1;
	        unsigned int newX;
	        if (x == INT_MIN) newX = static_cast<unsigned int>(INT_MAX) + 1;
	        else newX = static_cast<unsigned int>(x * sign);
	        string result;
	        while (newX != 0) {
	            result.append(1, newX % 10 + '0');
	            newX /= 10;
	        }
	        int now = 0;
	        unsigned int max = static_cast<unsigned int>(INT_MAX);
	        unsigned int min = max + 1;
	        for (int i = 0; i < result.size(); i++) {
	            if ((min - (result[i] - '0')) / 10 < now) return 0;
	            else if ((now * 10 + (result[i] - '0') == min) && sign == 1) return 0;
	            else        
	                now = now * 10 + (result[i] - '0');
	        }
	        return now * sign;
	    }
	};
```
&emsp;&emsp;然后程序便顺利AC了。
&emsp;&emsp;在讨论区也有类似的想法，使用long型来进行计算，long型的好处就是在类型转换的时候方便一些，代码如下：
```C++
	class Solution {
	public:
	    int reverse(int x) {
	        long num = abs((long)x);
	        long new_num = 0;
	        while(num) {
	            new_num = new_num*10 + num%10;
	            num /= 10;
	        }
	
	        if (new_num > INT_MAX) {
	            return 0;
	        }
	        return (x<0 ? -1*new_num : new_num);
	    }
	};
```
&emsp;&emsp;最后我又想到一个有意思的判断溢出的想法，想法来源于在整数溢出中，如果一个正数在运算越界后会得到一个无法确定的数，虽然我无法确定这个数是多少，但是我可以确定的是这个数一定不是我想得到的数，因此我只要将我的运算进行一下逆运算看看和运算之前的数一不一致就可以知道在运算过程中有没有发生越界。另外，发现将数字先转换成字符串再转回数字的过程完全是多余的，直接从数字到数字转换就可以了，代码如下：
```C++
	class Solution {
	public:
	    int reverse(int x) {
	        if (x == INT_MIN) return 0;
	        int newX = abs(x);
	        int result = 0;
	        while (newX != 0) {
	           if ((result * 10 + newX % 10 - newX % 10) / 10 != result) return 0;
	           result = result * 10 + newX % 10;
	           newX = newX / 10;
	        }
	        return x < 0 ? -result : result;
	    }
	};
```
&emsp;&emsp;程序顺利AC。
&emsp;&emsp;通过这道题其实可以发现，很多时候解决问题并不复杂，复杂的是发现问题，尤其是在发生错误之前将可能出现问题的都考虑到更复杂，希望以后可以考虑问题更加周全。
&emsp;&emsp;最后阅读了一些有关整数运算溢出的材料，有几个重要的讯息：


1. stl标准库中定义了INT_MAX和INT_MIN来表示int类型的最大最小值。
2. 显式类型转换
	```C++
	double d = 97.0;
	char ch = static_cast<char>(d);
	```
3. 什么是整数溢出？
&emsp;&emsp;既然一个整数是一个固定的长度(在本篇文章中使用32位)，它能存储的最大值是固定的，当尝试去存储一个大于这个固定的最大值时，将会导致一个整数溢出。在ISO C99的标准中讲到整数溢出将会导致"**不能确定的行为**"，也就是说编译器遵从了这个的规则，那就是完全忽略溢出而退出这个程序。很多编译器似乎忽略了这个溢出，结果是一个意想不到的错误值被存储。
4. 什么时候会发生整数溢出？
&emsp;&emsp;"A computation involving unsigned operands can never overflow, because a result that cannot be represented by the resulting unsigned integer type is reduced modulo the number that is one greater than the largest value that can be represented by the resulting type." 
&emsp;&emsp;当两个操作数都是有符号数时，溢出才有可能发生。而且溢出的结果是未定义的。当一个运算的结果发生溢出时，任何假设都是不安全的。例如，假定a和b是两个非负的整型变量(有符号)，我们需要检查a+b是否溢出，一种想当然的方式是:```if (a + b < 0)	overflow;```
&emsp;&emsp;实际上，在现实世界里，这并不能正常运行。当a+b确实发生溢出时，所有关于结果如何的假设均不可靠。比如，在某些机器的cpu，加法运算将设置一个内部寄存器为四种状态:正，负，零和溢出。在这种机器上，c编译器完全有理由实现以上的例子，使得a+b返回的不是负，而是这个内存寄存器的溢出状态。显然，if的判断会失败。
5. 为什么整数溢出危险的?
&emsp;&emsp;整数溢出是不能被立即察觉，因此没有办法去用一个应用程序来判断先前计算的结果是否实际上也是正确的。如果是用来计算缓冲区的大小或者计算数组索引排列的距离，这会变的危险。当然很多整数溢出并不是都是可利用的，因为并没有直接改写内存，但是有时，他们可导致其他类型的bugs，缓冲区溢出等。而且，整数溢出很难被发现，因此，就算是审核过的代码也会产生意外。 

####参考资料
1. http://bbs.chinaunix.net/thread-161810-1-1.html
   整数溢出与程序安全
2. http://blog.csdn.net/bichenggui/article/details/4734040
   如何判断整数溢出
###TODO
**学习C++ STL标准库的内容**
