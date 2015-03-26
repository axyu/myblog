title: "LeetCode刷题心得——TwoSum"
date: 2015-03-13 20:20:12
tags: [LeetCode,C/C++,hashmap]
---
###Two Sum
>Total Accepted: 69982 Total Submissions: 386997
>Given an array of integers, find two numbers such that they add up to a specific target number.

>The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

>You may assume that each input would have exactly one solution.

>**Input:** numbers={2, 7, 11, 15}, target=9 
>**Output:** index1=1, index2=2
<!--more-->

&emsp;&emsp;刚看到题目想到的解法是类似于冒泡排序的思想，使用两重循环寻找顺序容器中每一个数字是否可以与其余数字加和等于target，代码如下：

```c++
	class Solution {
	public:
	    vector<int> twoSum(vector<int> &numbers, int target) {
	        vector<int> result;
	        for (int i = 0; i < numbers.size() - 1; i++) {
	            for (int j = i + 1; j < numbers.size(); j++) {
	                if (numbers[i] + numbers[j] == target) {
	                    result.push_back(i + 1);
	                    result.push_back(j + 1);
	                    return result;
	                }
	            }
	        }
	    }
	};
```


&emsp;&emsp;提交上去发现提示**Time Limit Exceeded**错误，说明系统无法接受时间复杂度为O(n^2)的算法，因此考虑新的算法，通过再次观察题目可以发现，时间消耗较大的地方是当遍历容器中的数字`numbers[i]`时在剩余数字中查找`target-numbers[i]`的时候，时间复杂度为O(n)，因此考虑更快速的查找算法。

&emsp;&emsp;可以想到的快速的查找算法有HashMap，时间复杂度为O(1)以及排序之后的二分查找，时间复杂度为O(logn)。

&emsp;&emsp;首先实现HashMap查找算法，但是遇到一个问题，在C++中没有现成的HashMap库函数可以使用，便使用map代替，通过查找资料[1]发现map的一些特别之处，以`map<int, int> m`为例：

1.	在查找map时，如果使用下标方式查找，如`m[key]`，如果此时m中不存在key，此操作会自动在m中插入一个键值为key，值为类型初始值的一个pair，这样便改变了map的结构，因此不能使用，需要考虑使用count()和find()函数，这两个函数都不会造成多余的插入操作。
2.	map中的键值必须保证是唯一的，因此在插入同一个键值多次时，后插入的会覆盖前插入的，然而题目中的容器中会出现重复值，因此将map中的值类型改成`vector<int>`，这样可以保存相同值的多个位置。
3.	map中的find()函数是通过红黑树索引查找的，效率不比HashMap，具体细节未探明。

&emsp;&emsp;最终代码如下：
```c++
	class Solution {
	public:
	    static void myInsert(vector<int> &result, int pst1, int pst2) {
	        if (pst1 < pst2)  {
	            result.push_back(pst1);
	            result.push_back(pst2);
	        }
	        else {
	            result.push_back(pst2);
	            result.push_back(pst1);
	        }
	        return;
	    }
	    vector<int> twoSum(vector<int> &numbers, int target) {
	        vector<int> result;
	        map<int, vector<int>> numbersMap;
	        
	        for (int i = 0; i < numbers.size(); i++) {
	            vector<int> pst = numbersMap[numbers[i]];
	            pst.push_back(i + 1);
	            numbersMap[numbers[i]] = pst;
	        }
	        
	        for (map<int, vector<int>>::const_iterator map_it = numbersMap.begin(); map_it != numbersMap.end(); map_it++) {
	            map<int, vector<int>>::const_iterator it = numbersMap.find(target - map_it->first);
	            if (it != numbersMap.end()) {
	                if (map_it->first == target - map_it->first) {
	                    if (it->second.size() == 2)
	                        return it->second;
	                    else continue;
	                }
	                else
	                    myInsert(result, map_it->second[0], it->second[0]); 
	            }
	            else continue;
	        }
	        return result;
	    }
	};
```
&emsp;&emsp;结果取得了LeetCode中的第一个**AC**，感觉很好。紧接着实现先排序后查找的算法。

&emsp;&emsp;这个算法首先要解决的问题是在vector上的排序如何实现，自己写排序算法一来很费事，二来效率不一定很好，因此考虑使用现成排序算法，通过查阅资料[2]发现stl中在vector上已经设计了好了排序算法，我只需要给出排序的函数cmp即可，然后直接调用`sort(v.begin(), v.end(), cmp);`就可以实现对v的排序，很好很强大。排序解决之后，剩下的就好办了，在已经排好序（升序）的vector上我并没有简单地采用遍历+二分查找的传统方法，在这道题的设定下，可以直接通过创建首尾两个指针，分别从前向后和从后向前遍历vector，如果两个指针指的值之和大于target就将尾指针向前挪一位，同理如果两个指针所指值之和小于target就将首指针向后挪一位，直到寻找到满足条件的两个数为止。

&emsp;&emsp;除此之外，还发现了一个C++和C在定义结构体时的区别，以后要注意，C中在定义结构体时一般是:

```c++
	struct Node {
	    int value;
	    Node* point;
	};
```
&emsp;&emsp;然后定义新的结构体变量时，需要`struct Node node`;还要加上关键字`struct`，而在C++中可以省略这个关键字，方便了很多。

&emsp;&emsp;具体代码如下：
```c++
	class Solution {
	public:
	    struct Node {
	        int value;
	        int position;
	    };
	    
	    static bool cmp(Node num1, Node num2) {
	        return num1.value < num2.value;
	    }
	    
	    static void myPushBack(vector<int> &result, Node num1, Node num2) {
	        if (num1.position < num2.position) {
	            result.push_back(num1.position);
	            result.push_back(num2.position);
	        }
	        else {
	            result.push_back(num2.position);
	            result.push_back(num1.position);
	        }
	        return;
	    }
	    
	    vector<int> twoSum(vector<int> &numbers, int target) {
	        vector<int> result;
	        vector<Node> newNumbers;
	        
	        for (int i = 0; i < numbers.size(); i++) {
	            Node node;
	            node.value = numbers[i];
	            node.position = i + 1;
	            newNumbers.push_back(node);
	        }
	        
	        sort(newNumbers.begin(), newNumbers.end(), cmp);
	        
	        int begin = 0;
	        int end = newNumbers.size() - 1;
	        
	        while (begin < end) {
	            if (newNumbers[begin].value + newNumbers[end].value == target) {
	                myPushBack(result, newNumbers[begin], newNumbers[end]);
	                return result;
	            }
	            else if (newNumbers[begin].value + newNumbers[end].value < target) {
	                begin ++;
	            }
	            else {
	                end --;
	            }
	        }
	    }
	};
```
&emsp;&emsp;结果便获得了第二个**AC**。

###参考资料
1. http://www.linuxidc.com/Linux/2014-07/104619.htm 
   C++关联容器map 类型小结_Linux编程_Linux公社-Linux系统门户网站
2. http://blog.csdn.net/hnu_zxc/article/details/6746029 
   C++标准库vector排序 - hnuzxc的专栏 - 博客频道 - CSDN.NET




