## JAVA Grammer

1. JAVA封装类的比较不用==，而是用equals

```
Map<Character,Integer> mapS = new HashMap<>();
Map<Character,Integer> mapT = new HashMap<>();

mapS.get(ch).equals(mapT.get(ch))
```

## Hash table 哈希表

哈希表用于快速查找，在JAVA中，可以用hashSet和hashMap来直接作为哈希表。

哈希表用途1：改善稀疏的桶排序。 </br>
例如：给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。注意：若 s 和 t 中每个字符出现的次数都相同，则称 s 和 t 互为字母异位词。</br>
基本思路就是用桶排序来记录字符串中字符的分布情况，但桶数组的大小取决于`所有可能出现的字符集大小`，但如果用hashMap在运行时构建数组，那桶的大小取决于`实际出现的字符集大小`。

哈希表用途2：相较于数组，可以处理复杂情况和负数<br>
哈希表用途3: 可以充当记忆

例题： LeetCode1: 两数之和 https://leetcode.cn/problems/two-sum/description/

这道题要求找到数组中俩个数之和为target，并返回那两个数的坐标。 类比人类思维，如果人能记住之前的数字，那么当他每读到新的数字，可以立刻判断出这个数能否和之前的数构成target。而hashtable可以充当记忆，每读一个数就记到hashtable里，而判断 target-当前值 是否在hashtable只需要O(1)的实际。

哈希表用途4：和递归或者动态规划结合，将复杂度从n^n减到n^2

例题：leetcode454: https://leetcode.cn/problems/4sum-ii/submissions/


## 快慢指针
快慢指针常用于链表问题。

用途1：判断链表是否有环

Leecode202 快乐数： https://leetcode.cn/problems/happy-number/description/

该题思路就是： 如果是快乐数，当快指针到达1的时候，它将一直为1；这时慢指针会追上来。 如果不是快乐数，则可以看作是带环链表，快慢指针进入环后，快指针总是会追上慢指针。

另一种解法是将每一步放入hash set，用hash set来判断链表是否存在环。但是会有内存限制，快慢指针是速度有影响。

