# LeetCode算法题汇总
刷题顺序 https://blog.csdn.net/lnho2015/article/details/50962989

## 1. 两数之和(Two Sum)
地址： 

- [英文](https://leetcode.com/problems/two-sum/description/)
- [中文](https://leetcode-cn.com/problems/two-sum/description/)

### 1.1. 算法描述
Given an array of integers, return indices of the two numbers such that they add up to a specific target.You may assume that each input would have exactly one solution, and you may not use the same element twice.

> Example:
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].

### 1.2. 算法思路
1.  暴力求解。 时间复杂度为n(O^2)
2.  先排序，然后再使用两个指针左右左右夹逼(转换为剑指offer中那道题)。 复杂度是O(nlogn)
3.  使用hash表，先将数值保存在hash表中，hash表保存数值和下标的映射关系。遍历每个值，看到目标的差值是否在hash中，如果在表示找到。 复杂度为O(n)

### 1.3. 实现
#### java

```<java>
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] result = new int[2];
        if (nums == null || nums.length < 2) {
            return result;
        }
        
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int i = 0; i < nums.length; i++ ) {
            if (map.get(target - nums[i]) != null) {
                result[0] = map.get(target - nums[i]);
                result[1] = i;
            } else {
                map.put(nums[i], i);
            }
        }
        return result;
    }
}
```


## 8. String to Integer (atoi)
https://leetcode.com/problems/string-to-integer-atoi/description/

### 算法描述
### 算法思路
这道题就是要考虑一些特殊的情况：

1. 去除空格
2. 数字前出现多余的字符
3. 数字中有空格或非数字字符
4. 越界
5. 正负号问题

### 算法实现
https://www.cnblogs.com/springfor/p/3896499.html

```<java>
class Solution {
    public int myAtoi(String str) {
        // 1. null or empty string  
        if (str == null || str.length() == 0) return 0;  
          
        // 2. whitespaces  
        str = str.trim();  
        if (str.length() == 0) return 0;
          
        // 3. +/- sign  
        boolean positive = true;  
        int i = 0;  
        
        if (str.charAt(0) == '+') {  
            i++;  
        } else if (str.charAt(0) == '-') {  
            positive = false;  
            i++;  
        }  
          
        // 4. calculate real value  
        long tmp = 0;  
        for ( ; i < str.length(); i++) {  
            int digit = str.charAt(i) - '0';  
            if (digit < 0 || digit > 9) break;  
              
            // 5. handle min & max  
            if (positive) {  
                tmp = 10 * tmp + digit;  
                if (tmp > Integer.MAX_VALUE) return Integer.MAX_VALUE;  
            } else {  
                tmp = 10 * tmp - digit;  
                if (tmp < Integer.MIN_VALUE) return Integer.MIN_VALUE;  
            }  
        }  
          
        int ret = (int)tmp;  
        return ret; 
    }
}
```


## 15. 3Sum 
https://leetcode.com/problems/3sum/description/
### 算法描述
Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note:
The solution set must not contain duplicate triplets.

>Example:
>Given array nums = [-1, 0, 1, 2, -1, -4],
>A solution set is:
>[
>  [-1, 0, 1],
>  [-1, -1, 2]
>]


### 算法思路
1. 先排序，然后左右夹逼，排序之后，我们就可以对数组用两个指针分别从前后两端向中间扫描了，如果是 2Sum，我们找到两个指针之和为target就OK了，那 3Sum 类似，我们可以先固定一个数，然后找另外两个数之和为第一个数的相反数就可以了。 复杂度O(n^2)

2. 从头遍历数组，每次固定一个值x，去剩余部分找到另外两个数，使得他们的值刚好是当前值的相反数 -x。这个地方可以使用Two sum的思路，从里面找到一个和为特定值的数。 复杂度为O(n^2) 

这里采用第一种方式，第二种可以参考Two Sum， 只是外面多加一个for循环就可以了。

### 算法实现

```<java>
class Solution {
     public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> resultList = new ArrayList<>();
        if (null == nums || nums.length < 3) {
            return resultList;
        }

        // 排序
        quickSort(nums, 0, nums.length -1); // 或者使用 Arrays.sort(nums);

        // 两边夹逼
        int last = nums.length - 1;
        for (int i = 0; i <= last - 2; i++) {
            if (i > 0 && nums[i] == nums[i-1]) continue;
            find(nums, i + 1, last, nums[i], resultList); // 其实有点类似转为 2sum思路。
        }

        return resultList;
    }

    /**
     * 寻找两个数与num[i]的和为0
     */
    public void find(int[] nums, int begin, int end, int target, List<List<Integer>> resultList) {
        int l = begin, r = end;
        while (l < r) {
            if (nums[l] + nums[r] + target == 0) {
                List<Integer> ans = new ArrayList<Integer>();
                ans.add(target);
                ans.add(nums[l]);
                ans.add(nums[r]);
                resultList.add(ans); //放入结果集中
                while (l < r && nums[l] == nums[l+1]) l++;
                while (l < r && nums[r] == nums[r-1]) r--;
                l++;
                r--;
            } else if (nums[l] + nums[r] + target < 0) {
                l++;
            } else {
                r--;
            }
        }
    }

    private void quickSort(int[] array, int low, int high) {
        if (high < low) {
            return;
        }
        int lt = low, gt = high, i = low + 1;
        int val = array[low];
        while (i <= gt) {
            if (array[i] < val) {
                swap(array, i++, lt++);
            } else if (array[i] > val) {
                swap(array, i, gt--);
            } else {
                i ++;
            }
        }
        array[i - 1] = val;
        quickSort(array, low, lt -1);
        quickSort(array, gt + 1, high);
    }

    private void swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
}
```


方法二(解决不了重复数值问题)：-----还有问题没解决

```
class Solution {
     public List<List<Integer>> threeSum(int[] nums) {
         List<List<Integer>> result = new ArrayList<List<Integer>>();
         
         if (nums == null || nums.length < 3) return result;
         Map<Integer, Integer> map = new HashMap<Integer, Integer>();
         Arrays.sort(nums);
         for (int i = 0; i < nums.length; i++) {
             map.put(nums[i], i);
         }
         
          for (int i = 0; i <= nums.length - 2; i++) {
              if (i > 0 && nums[i] == nums[i-1] ) continue;
              findTwoSum(nums[i], i + 1, nums, map, result);
         }
         return result;
    }
    
    public void findTwoSum(int target, int begin, int[] nums, Map<Integer, Integer> map,  List<List<Integer>> result) {
        int reVal = -1 * target;
        for (int i = begin; i < nums.length; i++) {
            if (nums[i] == nums[i-1])  continue;
            
            Integer val = map.get(reVal - nums[i]);
            if (val != null && val > i) {
                List<Integer> temp = new ArrayList<Integer>();
                temp.add(target);
                temp.add(nums[i]);
                temp.add(reVal - nums[i]);
                result.add(temp);
                
            } 
        }
    }
}
```



## 20. Valid Parentheses 
https://leetcode.com/problems/valid-parentheses/submissions/1
### 算法描述
Given a string containing just the characters , determine if the input string is valid.

The brackets must close in the correct order, “()” and “()[]{}” are all valid but “(]” and “([)]” are not.

给定一个字符串，只包含’(‘, ‘)’, ‘{‘, ‘}’, ‘[’ 和’]’这些字符，检查它是否是“有效”的。 
括号必须以正确的顺序关闭，例如”()” 和”()[]{}”都是有效的，”(]” 和”([)]”是无效的。

### 算法思想
本题考查的是栈结构，具有后进先出的特性。有效包含2个方面，第一个是如果是关闭的括号，前一位一定要刚好有一个开启的括号；第二个是最终结果，需要把所有开启的括号都抵消完。一个比较容易出错的地方是，遇到关闭括号时，判断栈是否已经空或栈顶元素是对应的开启符号。

### 算法实现

```<java>
public class Solution {
    public boolean isValid(String s) {
        Map<Character,Character> pairs = new HashMap<Character,Character>();
        pairs.put('}','{');
        pairs.put(')','(');
        pairs.put(']','[');
        
        Stack<Character> stack = new Stack<Character>();
        char[] charArray = s.toCharArray();
        for(char cVal : charArray){
            if(pairs.containsKey(cVal)){
                if(stack.isEmpty() || pairs.get(cVal) != stack.pop())
                    return false;
            }else if(!pairs.containsKey(cVal) ){
                stack.push(cVal);
            }
        }
        return  stack.isEmpty();
    }
}
```

## 21. 合并两个有序链表(Merge Two Sorted Lists)
地址:

- [英文](https://leetcode.com/problems/merge-two-sorted-lists/description/)
- [中文](https://leetcode-cn.com/problems/merge-two-sorted-lists/description/)


### 21.1 算法描述
#### 英文
Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

Example:

Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
#### 中文
将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4

### 21.2 算法思路

合并算法

### 21.3 算法实现

```<java>
public class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if(l1 == null) return l2;
        if(l2 == null) return l1;
        ListNode head;
        if(l1.val < l2.val){
            head = l1;
            head.next = mergeTwoLists(l1.next,l2);
        }else{
            head = l2;
            head.next = mergeTwoLists(l1,l2.next);
        }
        return head;
    }
}
```


## 28. 实现strStr()(Implement strStr())---字符串匹配
- [英文版地址](https://leetcode.com/problems/implement-strstr/description/)
- [中文版地址]()

### 28.1 算法描述
实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:

>输入: haystack = "hello", needle = "ll"
>输出: 2

示例 2:

>输入: haystack = "aaaaa", needle = "bba"
>输出: -1

说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

### 28.2 算法思路
采用KMP算法

需要注意特殊输入：

- “”和“”
- “dda”有值和无值 “”

### 28.3 算法实现

```<java>
class Solution {
    public int strStr(String haystack, String needle) {
        if (haystack == null || needle == null) return -1;
        if (needle.length() == 0) return 0;
    
        char[] pattern = needle.toCharArray();
        char[] str = haystack.toCharArray();
        int[] next = makeNext(pattern);
        for (int strIdx = 0, pIdx = 0; strIdx < str.length; ++strIdx) {
            while (pIdx > 0 && pattern[pIdx] != str[strIdx]) {
                // 移动匹配字符串， 这边需要-1。  
                pIdx = next[pIdx - 1];
            }
            if (pattern[pIdx] == str[strIdx]) {
                pIdx ++;
            }
            if (pIdx == pattern.length) {
                return strIdx - pattern.length + 1;
            }
        }
        return -1;
    }
         
    // 计算部分匹配表
    public int[] makeNext(char[] pattern) {
        int[] next = new int[pattern.length];
        int maxSuffixLen = 0;  // maxSuffixLen 最大前后缀长度
        next[0] = 0; // 模板字符串的第一个字符的最大前缀长度为0
        
        // idx 模板字符串下标，
        for (int idx = 1; idx < pattern.length; idx ++) {
            // maxSuffixLen 大于0 表示前一个字符已经存在匹配
            while(maxSuffixLen > 0 && pattern[idx] != pattern[maxSuffixLen]) {
                // 递归求出最大的相同前后缀长度
                maxSuffixLen = next[maxSuffixLen -1 ];
            }
            
            if (pattern[idx] == pattern[maxSuffixLen]) {
                // 如果相等，那么最大相同前后缀长度加1
                maxSuffixLen++;
            }
            next[idx] = maxSuffixLen;
        }
        return next;
    }
}
```

## 56. 合并区间（Merge Intervals）
- [英文版本地址](https://leetcode.com/problems/merge-intervals/description/)
- [中文版本地址](https://leetcode-cn.com/problems/merge-intervals/description/)

### 56.1 算法描述

给出一个区间的集合，请合并所有重叠的区间。

示例 1:

>输入: [[1,3],[2,6],[8,10],[15,18]]
>输出: [[1,6],[8,10],[15,18]]
>解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].

示例 2:

>输入: [[1,4],[4,5]]
>输出: [[1,5]]
>解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。


### 56.2 算法思路
这道题目给了我们一个区间的list，让我们返回一个list，是合并了所有有重叠的区间之后的list。这道题目的关键在于如何判断两个区间有重叠，根据原题给的例子可以看出，在按照每个区间的start排序之后很容易判断出是否相邻的两个区间有交集。我们看第一个区间[1,3] 中的end 3 > 第二个区间[2,6] 的start 2。在排序完之后，只要前面一个区间的end 大于等于 后面一个区间的start，它们就是重叠的。


那么为了merge 这两个区间，保留第一个区间的start， 在两个end里面拿大的。设一个temp 等于第一个区间，遍历剩下的区间。然后每次拿temp 和这个区间比较，要注意的是，如果遇到了重叠的，把temp 更新为merge 后的区间，这时不需要加入list，因为还有可能继续和下一个区间重叠。当temp和这个区间不重叠的时候，把temp 加入list，再把这个区间设为新的temp。


### 56.3 算法实现

```<java>
public List<Interval> merge(List<Interval> intervals) {
    if (intervals.size() <= 1)
        return intervals;
    
    // Sort by ascending starting point using an anonymous Comparator
    intervals.sort((i1, i2) -> Integer.compare(i1.start, i2.start));
    
    List<Interval> result = new LinkedList<Interval>();
    int start = intervals.get(0).start;
    int end = intervals.get(0).end;
    
    for (Interval interval : intervals) {
        if (interval.start <= end) // Overlapping intervals, move the end if needed
            end = Math.max(end, interval.end);
        else {                     // Disjoint intervals, add the previous one and reset bounds
            result.add(new Interval(start, end));
            start = interval.start;
            end = interval.end;
        }
    }
    
    // Add the last interval
    result.add(new Interval(start, end));
    return result;
}
```

