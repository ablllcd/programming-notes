## JAVA Grammer

1. JAVA封装类的比较不用==，而是用equals

```
Map<Character,Integer> mapS = new HashMap<>();
Map<Character,Integer> mapT = new HashMap<>();

mapS.get(ch).equals(mapT.get(ch))
```

2. Set类如何一边遍历一边删除值

不能用key:map.keySet()这种强循环，必须要用迭代器，而且对元素的删除也必须由迭代器完成，不能用map.remove(key)。

```
while(map.size()>k){
    // 对每个元素的频率都减一，频率为0就删除
    Iterator<Integer> iterator = map.keySet().iterator();
    while(iterator.hasNext()){
        int key = iterator.next();
        int value = map.get(key);
        if(value == 1){
            iterator.remove();
        }else{
            map.replace(key,value-1);
        }
    }
}
```

## 数组

### 二分查找

情节：在数组中搜索目标

前提：数组需要是有序的

思路：每次取数组中间值和目标值比较，然后判断搜索左半边还是右半边。

例题： https://leetcode.cn/problems/binary-search/


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

哈希表缺点1：当问题复杂时，代码会难以书写。而且无法直接对列表去重，例如 [1,2] 和 [2,1] 会被当作不同的元素。

例题：

## 快慢指针
快慢指针常用于链表问题。

**用途1**：判断链表是否有环

Leecode202 快乐数： https://leetcode.cn/problems/happy-number/description/

该题思路就是： 如果是快乐数，当快指针到达1的时候，它将一直为1；这时慢指针会追上来。 如果不是快乐数，则可以看作是带环链表，快慢指针进入环后，快指针总是会追上慢指针。

另一种解法是将每一步放入hash set，用hash set来判断链表是否存在环。但是会有内存限制，快慢指针是速度有影响。

**用途2**：和排序合作找到目标值

将数组排序后，可以用两个指针从左右两个方向逼近筛选出目标值。


## 字符串

java中字符串是不可修改的

字符串split如果要移除空白，split(" ")并不好用，因为如果有两个space则无法处理，应该正则表达式\s+。

**类型1 反转字符串**：https://github.com/youngyangyang04/leetcode-master/blob/master/problems/0344.%E5%8F%8D%E8%BD%AC%E5%AD%97%E7%AC%A6%E4%B8%B2.md

定义左右两个指针，一边靠近一边交换即可

**类型2 反转单词**：https://leetcode.cn/problems/reverse-words-in-a-string/

如果需要O(1)的额外空间，可以先反转整个字符串，然后在反转每个单词，只是时间复杂度会增加，需要读多次。

例子："ni hao" -> "oah in" -> "hao ni"

## 树

树总是用**栈/队列**来进行遍历，例如

1. 树的前序/中序/后序遍历的循环实现：用stack来模拟递归
2. 树的层序遍历：用queue来存储节点
3. 反转二叉树：用stack或者queue来读取每层（顺序无关）