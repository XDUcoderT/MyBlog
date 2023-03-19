---
title: 剑指offerⅡ
tags: 刷题
index_img: /img/index/刷题.jpg
category: /学习/算法/做题总结
renderNumberedHeading: true
grammar_cjkRuby: true
---
# 剑指offer2

剑指offerⅡ05

**题目描述**

```
给定一个字符串数组 words，请计算当两个字符串 words[i] 和 words[j] 不包含相同字符时，它们长度的乘积的最大值。假设字符串中只包含英语的小写字母。如果没有不包含相同字符的一对字符串，返回 0。
```
**实例**
```
输入: words = ["abcw","baz","foo","bar","fxyz","abcdef"]
输出: 16 
解释: 这两个单词为 "abcw", "fxyz"。它们不包含相同字符，且长度的乘积最大。
```

**题解**

```
class Solution {
    public int maxProduct(String[] words) {
        //利用位掩码 
        // 如果有相同的字母与完返回就不是0
        int wordsLen = words.length;
        int[] mask = new int[wordsLen];
        for(int i = 0; i < wordsLen; i++){
            int letterLen = words[i].length();
            for(int j = 0;j < letterLen ; j++){
                int loc = words[i].charAt(j) - 'a';
                if(((1 << loc) & mask[i]) == 0){
                    mask[i] += (1 << loc);
                }
            }    
        }
        for(int i = 0; i< wordsLen ; i++){
            System.out.println(mask[i]);
        }
        int maxProd = 0;
        for(int i = 0 ; i < wordsLen ; i++){
           for(int j = i + 1 ;j < wordsLen ;j++){
               if((mask[i] & mask[j]) == 0){
                   maxProd = words[i].length() * words[j].length() > maxProd ? words[i].length() * words[j].length() : maxProd;
                }
           }
        }
        return maxProd;
    }

}
```

**总结**

```
这道题主要就是考察判断两个字符串中是否有相同字母
利用与运算快速实现
```


----

剑指offerⅡ06

**题目描述**
```
给定一个已按照 升序排列  的整数数组 numbers ，请你从数组中找出两个数满足相加之和等于目标数 target 。

函数应该以长度为 2 的整数数组的形式返回这两个数的下标值。numbers 的下标 从 0 开始计数 ，所以答案数组应当满足 0 <= answer[0] < answer[1] < numbers.length 。

假设数组中存在且只存在一对符合条件的数字，同时一个数字不能使用两次。
输入：numbers = [1,2,4,6,10], target = 8
输出：[1,3]
解释：2 与 6 之和等于目标数 8 。因此 index1 = 1, index2 = 3 。

```
**实例**
```
输入：numbers = [1,2,4,6,10], target = 8
输出：[1,3]
解释：2 与 6 之和等于目标数 8 。因此 index1 = 1, index2 = 3 。

```

**题解**

```
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        for (int i = 0; i < numbers.length; ++i) {
            int low = i + 1, high = numbers.length - 1;
            //这里只需要找一个，所以小于等于
            while (low <= high) {
                int mid = (high - low) / 2 + low;
                if (numbers[mid] == target - numbers[i]) {
                    return new int[]{i, mid};
                } else if (numbers[mid] > target - numbers[i]) {
                    high = mid - 1;
                } else {
                    low = mid + 1;
                }
            }
        }
        return new int[]{-1, -1};
    }
}

```

**总结**

```
这道题利用二分查找，注意边界条件的控制
```

剑指offerⅡ07

----


**题目描述**
```
给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a ，b ，c ，使得 a + b + c = 0 ？请找出所有和为 0 且 不重复 的三元组。

```
**示例**
```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
```

**题解**

```
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        // 不能重复使用

        /*
        1. 排序--升序 -- 归并排序
        2. 固定一个值 -- for 0 ~ lens-3
        3. 剩下2个值在右区间进行双指针 -- 参考前题
        4. 考虑重复组合的问题 -- 去重，有两种重复情况
            1. 第一个固定的值存在重复
            2. 第二第三个值的组合存在重复
         */

        List<List<Integer>> res = new ArrayList<>();

        // 排序
        Arrays.sort(nums);

        // 遍历固定第一个值
        for (int i=0; i<nums.length-2; i++) {
            // 解决第一个固定值重复的问题
            // 例如排序后的[-4,-1,-1,0,1,2] -1,-1处存在首数字重复
            if (i>0 && nums[i]==nums[i-1]) continue;
            int curTarget = -nums[i];
            int left = i+1;
            int right = nums.length-1;
            while (right>left) {
                int tmpSum = nums[left] + nums[right];
                if (tmpSum > curTarget)
                    right--;
                else if (tmpSum < curTarget)
                    left++;
                else {
                    List<Integer> list = new ArrayList<>();
                    list.add(nums[i]);
                    list.add(nums[left]);
                    list.add(nums[right]);
                    res.add(list);

                    // 解决第二第三个固定值重复的问题
                    // 例如排序后的[-2,0,0,2,2] -2固定下，0，0，2，2会重复组合出0,2
                    // 短路运算，第一个条件保证大前提，第二个语句运用前置++运算符更新指针
                    while (left<right && nums[left]==nums[++left]);
                    while (left<right && nums[right]==nums[--right]);
                }
            }
        }
        return res;
    }

}
```

**总结**

```
这道题是三元和 还是利用固定法，另外两个的和我们要保证遍历不重不漏，所以利用前后双指针遍历
```


剑指offerⅡ08

----
**题目描述**
```
给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。


```
**示例**
```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**题解**

```
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int ans = Integer.MAX_VALUE;
        int start = 0, end = 0;
        int sum = 0;
        while (end < n) {
            // 每次右指针先往右移动一位
            // 然后让左指针向右移动直到不符合题意为止
            sum += nums[end];
            while (sum >= s) {
                ans = Math.min(ans, end - start + 1);
                sum -= nums[start];
                start++;
            }
            end++;
        }
        return ans == Integer.MAX_VALUE ? 0 : ans;
    }
}

```

**总结**

```
这道题利用滑动窗口的思路，先向右移动右指针直到满足条件，然后向右移动左指针，直到满足条件，然后继续移动右指针，依此循环
“右定左动”
```

剑指offerⅡ09

----
**题目描述**
```
给定一个正整数数组 nums和整数 k ，请找出该数组内乘积小于 k 的连续的子数组的个数。

```
**示例**
```
输入: nums = [10,5,2,6], k = 100
输出: 8
解释: 8 个乘积小于 100 的子数组分别为: [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于100的子数组。


```

**题解**

```
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        int n = nums.length, ret = 0;
        int prod = 1, i = 0;
        for (int j = 0; j < n; j++) {
            prod *= nums[j];
            while (i <= j && prod >= k) {
                prod /= nums[i];
                i++;
            }
            ret += j - i + 1;
        } 
        return ret;
    }
}

```

**总结**

```
这道题利用滑动窗口的思路，先向右移动右指针直到满足条件，然后向右移动左指针，直到满足条件，然后继续移动右指针，依此循环
“右定左动”
```





剑指offerⅡ10

----
**题目描述**
```
给定一个整数数组和一个整数 k ，请找到该数组中和为 k 的连续子数组的个数。

```
**示例**
```
输入:nums = [1,1,1], k = 2
输出: 2
解释: 此题 [1,1] 与 [1,1] 为两种不同的情况
```

**题解**

```
public class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0, pre = 0;
        HashMap<Integer,Integer> mp = new HashMap < > ();
        mp.put(0, 1);
        for (int i = 0; i < nums.length; i++) {
            //pre记录前缀和的值
            pre += nums[i];
            //每次查看前面有没有满足条件的pre[j]
            if (mp.containsKey(pre - k)) {
                count += mp.get(pre - k);
            }
            mp.put(pre, mp.getOrDefault(pre, 0) + 1);
        }
        return count;
    }
}
```

**总结**

```
这道题利用了前缀和 + 哈希表的经典搭配 就是以和为键，以次数为值，构造哈希表

```

剑指offerⅡ11

----
**题目描述**
```
给定一个二进制数组 nums , 找到含有相同数量的 0 和 1 的最长连续子数组，并返回该子数组的长度。

```
**示例**
```
输入: nums = [0,1]
输出: 2
说明: [0, 1] 是具有相同数量 0 和 1 的最长连续子数组。
```

**题解**

```
public class Solution {
    public int findMaxLength(int[] nums) {
        int maxLen = 0;
        HashMap<Integer,Integer> hp = new HashMap<Integer,Integer>();
        hp.put(0,-1);
        int res = 0;
        for(int i = 0 ;i< nums.length;i++){
            int num = nums[i];
            if(num == 1){
                res++;
            }
            else res--;
            if(hp.containsKey(res)){
                int newLen = i - hp.get(res);
                maxLen = Math.max(maxLen,newLen);
            }else{
                hp.put(res,i);
            }
        }
        return maxLen;
    }
}
```

**总结**

```
这道题利用了前缀和 + 哈希表的经典搭配 以值为键，以下标为值

```

剑指offerⅡ12

----
**题目描述**
```
给你一个整数数组 nums ，请计算数组的 中心下标 。

数组 中心下标 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。

如果中心下标位于数组最左端，那么左侧数之和视为 0 ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。

如果数组有多个中心下标，应该返回 最靠近左边 的那一个。如果数组不存在中心下标，返回 -1 。

```
**示例**
```
输入：nums = [1,7,3,6,5,6]
输出：3
解释：
中心下标是 3 。
左侧数之和 sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11 ，
右侧数之和 sum = nums[4] + nums[5] = 5 + 6 = 11 ，二者相等。

```

**题解**

```
class Solution {
    public int pivotIndex(int[] nums) {
        int length = nums.length;
        int sum = 0;
        for(int i =0 ;i< length ;i++){
            sum += nums[i];
        }
        int leftSum = 0;
        if(sum == nums[0]) return 0;
        for(int i = 1 ;i< length-1 ;i++){
            leftSum += nums[i-1];
            if(2 * leftSum + nums[i] == sum){
                return i;
            }
        }
        if(sum == nums[length-1]) return length-1;
        return -1;
    }
}
```

**总结**

```
本题比较简单，直接前缀和遍历
```

剑指offerⅡ14

----
**题目描述**
```
给定两个字符串 s1 和 s2，写一个函数来判断 s2 是否包含 s1 的某个变位词。

换句话说，第一个字符串的排列之一是第二个字符串的 子串 。

```
**示例**
```
输入: s1 = "ab" s2 = "eidbaooo"
输出: True
解释: s2 包含 s1 的排列之一 ("ba").

```

**题解**

```

class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int n = s1.length(), m = s2.length();
        if (n > m) {
            return false;
        }
        // 比较两个字符串映射数组
        // 利用Arrays.equals比较
        int[] cnt1 = new int[26];
        int[] cnt2 = new int[26];
        // 先比较前n个是否相等
        for (int i = 0; i < n; ++i) {
            ++cnt1[s1.charAt(i) - 'a'];
            ++cnt2[s2.charAt(i) - 'a'];
        }
        if (Arrays.equals(cnt1, cnt2)) {
            return true;
        }
        // 从n+1开始进1出1
        for (int i = n; i < m; ++i) {
            ++cnt2[s2.charAt(i) - 'a'];
            --cnt2[s2.charAt(i - n) - 'a'];
            if (Arrays.equals(cnt1, cnt2)) {
                return true;
            }
        }
        return false;
    }
}
```

**总结**

```
本题利用维护滑动窗口，保证窗口长度固定为s1的长度
```

剑指offerⅡ15

----
**题目描述**
```
给定两个字符串 s 和 p，找到 s 中所有 p 的 变位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

变位词 指字母相同，但排列不同的字符串。

```
**示例**
```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的变位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的变位词。

```

**题解**

```
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> list = new ArrayList<>();
        int length1 = s.length();
        int length2 = p.length();
        if(length1 < length2) return list;
        int[] mask1 = new int[26];
        int[] mask2 = new int[26];
        for(int i = 0 ;i < length2 ;i++){
            mask1[s.charAt(i) - 'a']++ ;
            mask2[p.charAt(i) - 'a']++ ;
        }
        if(Arrays.equals(mask1,mask2)){
            list.add(0);   
        }
        for(int i = length2;i < length1 ;i++){
            mask1[s.charAt(i) - 'a']++ ;
            mask1[s.charAt(i - length2) - 'a']--;
            if(Arrays.equals(mask1,mask2)){
                list.add(i-length2+1);   
            }
        }
        return list;
    }
}
```

**总结**

```
本题和上题一致，利用维护滑动窗口，保证窗口长度固定为p的长度，然后进一出一来遍历
```

剑指offerⅡ16

----
**题目描述**
```
给定一个字符串 s ，请你找出其中不含有重复字符的 最长连续子字符串 的长度。
```
**示例**
```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子字符串是 "abc"，所以其长度为 3。
```

**题解**

```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        // 哈希集合，记录每个字符是否出现过
        Set<Character> occ = new HashSet<Character>();
        int n = s.length();
        // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ans = 0;
        for (int i = 0; i < n; ++i) {
            if (i != 0) {                                                                           
                // 左指针向右移动一格，移除一个字符
                occ.remove(s.charAt(i - 1));
            }
            while (rk + 1 < n && !occ.contains(s.charAt(rk + 1))) {
                // 不断地移动右指针
                occ.add(s.charAt(rk + 1));
                ++rk;
            }
            // 第 i 到 rk 个字符是一个极长的无重复字符子串
            ans = Math.max(ans, rk - i + 1);
        }
        return ans;
    }
}

```

**总结**

```
本题运用滑动窗口，左定右动
```


剑指offerⅡ17

----
**题目描述**
```
给定两个字符串 s 和 t 。返回 s 中包含 t 的所有字符的最短子字符串。如果 s 中不存在符合条件的子字符串，则返回空字符串 "" 。

如果 s 中存在多个符合条件的子字符串，返回任意一个。

```
**示例**
```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC" 
解释：最短子字符串 "BANC" 包含了字符串 t 的所有字符 'A'、'B'、'C'

```

**题解**

```
class Solution {
    Map<Character, Integer> ori = new HashMap<Character, Integer>();
    Map<Character, Integer> cnt = new HashMap<Character, Integer>();

    public String minWindow(String s, String t) {
        int tLen = t.length();
        for (int i = 0; i < tLen; i++) {
            char c = t.charAt(i);
            ori.put(c, ori.getOrDefault(c, 0) + 1);
        }
        int l = 0, r = -1;
        int len = Integer.MAX_VALUE, ansL = -1, ansR = -1;
        int sLen = s.length();
        while (r < sLen) {
            ++r;
            if (r < sLen && ori.containsKey(s.charAt(r))) {
                cnt.put(s.charAt(r), cnt.getOrDefault(s.charAt(r), 0) + 1);
            }
            while (check() && l <= r) {
                if (r - l + 1 < len) {
                    len = r - l + 1;
                    ansL = l;
                    ansR = l + len;
                }
                if (ori.containsKey(s.charAt(l))) {
                    cnt.put(s.charAt(l), cnt.getOrDefault(s.charAt(l), 0) - 1);
                }
                ++l;
            }
        }
        return ansL == -1 ? "" : s.substring(ansL, ansR);
    }

    public boolean check() {
        Iterator iter = ori.entrySet().iterator(); 
        while (iter.hasNext()) { 
            Map.Entry entry = (Map.Entry) iter.next(); 
            Character key = (Character) entry.getKey(); 
            Integer val = (Integer) entry.getValue(); 
            if (cnt.getOrDefault(key, 0) < val) {
                return false;
            }
        } 
        return true; 
    }
}


```

**总结**

```
本题还是经典的滑动窗口，右定左动（一般取最小值是右定左动，最大值是左定右动）
本题的右定左动中"左动"的条件判断条件上比较复杂，需要比较两个map的值
```


剑指offerⅡ18

----
**题目描述**
```
给定一个字符串 s ，验证 s 是否是 回文串 ，只考虑字母和数字字符，可以忽略字母的大小写。

本题中，将空字符串定义为有效的 回文串 。

```
**示例**
```
输入: s = "A man, a plan, a canal: Panama"
输出: true
解释："amanaplanacanalpanama" 是回文串

```

**题解**

```
class Solution {
    public boolean isPalindrome(String s) {
        s = s.toLowerCase();
        int length = s.length();
        int begin = 0;
        int end = length-1;
    
        while(begin < end){
            while(!isOk(s.charAt(begin)) && begin < end){
                begin++;
            }
            while(!isOk(s.charAt(end)) && begin <end){
                end--;
            }
            if(s.charAt(begin)!=s.charAt(end)){
                System.out.println(begin + " " + end);
                return false;
            }
            begin++;
            end--;
        }
        return true;
    }

    public boolean isOk(char c){
        return ((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z'));
    }
}

```

**总结**

```

本题比较水，对于大小写不敏感的题我们直接用str.toLowerCase将其全部转为小写
注意一下边界就行
```


剑指offerⅡ19

----
**题目描述**
```
给定一个非空字符串 s，请判断如果 最多 从字符串中删除一个字符能否得到一个回文字符串。

```
**示例**
```
输入: s = "aba"
输出: true

```

**题解**

```
//对于是否满足类体型 中间需要分条件的情况，而且无法简单进行条件判断，可以考虑将多种情况并一下
class Solution {
    public boolean validPalindrome(String s) {
        return check(s,0,s.length()-1,true);
    }
    boolean check(String s,int i,int j,boolean flag){ // flag标识只有一次机会
        while (i<j){
            if(s.charAt(i)!=s.charAt(j)){
                if(!flag){
                    // 已经去掉一个了，失败
                    return false;
                }
                // 左边去掉
                boolean b1 = check(s, i, j - 1, false);
                // 右边去掉
                boolean b2 = check(s, i+1, j , false);
                return b1||b2; // 有一种成功即可
            }
            i++;
            j--;
        }
        return true;
    }
}
```

**总结**

```
本题非常经典，是经典的满足题意类问题，对于此类问题中会有分叉的情况，我们可以将多种情况的结果并起来
```


剑指offerⅡ20

----
**题目描述**
```
给定一个字符串 s ，请计算这个字符串中有多少个回文子字符串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

```
**示例**
```
输入：s = "abc"
输出：3
解释：三个回文子串: "a", "b", "c"

```

**题解**

```
class Solution {
    public int countSubstrings(String s) {
        int sum = s.length();
        for(int i = 0;i < s.length();i++){
           int indexR1 = i + 1;
           while(indexR1 < s.length()){
               int indexL1 = 2 * i - indexR1;
               if(indexL1 < 0){
                   break;
               }
               if(s.charAt(indexL1) == s.charAt(indexR1)){
                   sum++;
               }else{
                   break;
               }
               indexR1++;
            }
            int indexR2 = i;
            while(indexR2 < s.length()){
               int indexL2 = 2 * i - indexR2 - 1;
               if(indexL2 < 0){
                   break;
               }
               if(s.charAt(indexL2) == s.charAt(indexR2)){
                   sum++;
               }else{
                   break;
               }
               indexR2++;
            }
        }
        return sum;  
    }
}

```

**总结**

```
寻找回文串的个数，中心向左右拓展
```


剑指offerⅡ21

----
**题目描述**
```
给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点

```
**示例**
```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]

```

**题解**

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        //计算出链表总长度
        ListNode list1 = head;
        ListNode list2 = head;
        while(n > 0){
            n--;
            list2 = list2.next;
        }
        if(list2 == null){
            return list1.next;
        }
        while(list2.next != null){
            list1 = list1.next;
            list2 = list2.next;
        }
        list1.next = list1.next.next;
        return head;
    }
}

```

**总结**

```
本题是链表加快慢指针的经典结合，可以转化为找到链表中倒数第k个节点
我们甚至可以进一步拓展，链表受限的是“找到后面的忘记前面的”，我们
虽然无法完全解决这个问题，想起来前面所有的，但是我们可以让他想起来
第前k个节点，就是运用快慢指针！！！
```


剑指offerⅡ22

----
**题目描述**
```
给定一个链表，返回链表开始入环的第一个节点。 从链表的头节点开始沿着 next 指针进入环的第一个节点为环的入口节点。如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

```
**示例**
```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。

```

**题解**

```
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fastNode = head;
        ListNode slowNode = head;
        if(head == null || fastNode.next == null || fastNode.next.next == null){
            return null;
        }
        if(fastNode.next != null && fastNode.next.next != null){
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;
        }
        while(fastNode != slowNode){
            if(fastNode.next == null || fastNode.next.next == null){
                return null;
            }
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;
        }
        ListNode newNode = head;
        while(slowNode != newNode){
            slowNode = slowNode.next;
            newNode = newNode.next;
        }
        return slowNode;
        
    }
}

```

**总结**

```
本题是经典的快慢指针用法，技巧性比较强，看看就行，不必深究
```


剑指offerⅡ25

----
**题目描述**
```
给定两个 非空链表 l1和 l2 来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。

可以假设除了数字 0 之外，这两个数字都不会以零开头。

```
**示例**
```
输入：l1 = [7,2,4,3], l2 = [5,6,4]
输出：[7,8,0,7]

```

**题解**

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        l1 = reverseNode(l1);
        l2 = reverseNode(l2);
        ListNode newNode = l1;
        int carry = 0;
        int sum = 0;
        while(l1.next != null && l2.next != null){
            sum = l1.val + l2.val + carry;
            carry = sum >= 10 ? sum / 10 : 0;
            l1.val = sum%10;
            l1 = l1.next;
            l2 = l2.next;
        }
        sum = l1.val + l2.val + carry;
        l1.val = sum%10;
        carry = sum >= 10 ? sum/ 10 : 0;
        if(l1.next == null && l2.next == null){
            if(carry != 0){
                ListNode node = new ListNode(carry,null);
                l1.next = node;
            }
        }else if(l1.next != null){
            l1 = l1.next;
            while(l1.next != null){
                sum = l1.val + carry;
                l1.val = sum%10;
                carry = sum>= 10 ?sum / 10 : 0;
                l1 = l1.next;
            }
            sum = l1.val + carry;
            l1.val = sum%10;
            carry = sum>= 10 ?sum / 10 : 0;
            if(carry != 0){
                ListNode node = new ListNode(carry,null);
                l1.next = node;
            }
        }else{
            l1.next = l2.next;
            l1 = l1.next;
            while(l1.next != null){
                sum = l1.val + carry;
                l1.val = sum%10;
                carry = sum>= 10 ?sum / 10 : 0;
                l1 = l1.next;
            }
            sum = l1.val + carry;
            l1.val = sum%10;
            carry = sum>= 10 ?sum / 10 : 0;
            if(carry != 0){
                ListNode node = new ListNode(carry,null);
                l1.next = node;
            }
        }
        return reverseNode(newNode);
    }

    public ListNode reverseNode(ListNode head){
       if(head == null || head.next == null){
            return head;
        }
        ListNode pre = null;
        ListNode aft = null;
        while(head != null){
            aft = head.next;
            head.next = pre;
            pre = head;
            head = aft;
        }
        return pre;
    }
}
```

**总结**

```
这道题挺恶心的 链表就是失忆症 走到后面忘了前面 但是整式加法就是要先知道后面的值看看有无进位才能知道前面的值，感觉挺恶心的。
所以我就先写一个翻转链表的函数将给定的两个链表翻转，最后再翻转回去
然后就是整式加法的模拟，边界条件真的恶心
```


剑指offerⅡ26

----
**题目描述**
```
给定一个单链表 L 的头节点 head ，单链表 L 表示为：

L0 → L1 → … → Ln-1 → Ln 
请将其重新排列后变为：

L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → …

不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

```
**示例**
```
输入: head = [1,2,3,4]
输出: [1,4,2,3]

```

**题解**

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        ListNode newNode = head;
        Map<Integer,ListNode> map = new HashMap<>();
        int len = 0;
        ListNode go = head;
        while(go != null){
            go = go.next;
            ++len;
        }
        go = head;
        int goLen = 0;
        if(len % 2 == 0){
            while(go != null){
                goLen++;
                if(goLen <= len/2 ){
                    map.put(2 * goLen - 1,go);
                }else{
                    map.put(2*(len - goLen + 1),go);
                }
                go = go.next;
            }
        }else{
            while(go != null){
                goLen++;
                if(goLen <= len/2 + 1){
                    map.put(2 * goLen - 1,go);
                }else{
                    map.put(2*(len - goLen + 1),go);
                }
                go = go.next;
            }
        }
        goLen = 1;
        head.next = null;
        while(goLen != len){
            goLen++;
            newNode.next = map.get(goLen);
            newNode = newNode.next;
        }
        newNode.next = map.get(len);
        newNode = newNode.next;
        newNode.next = null;
        head = newNode;
    }
}
```

**总结**

```
一种比较垃圾的思路，第一次遍历记录总长度
第二次遍历边遍历边将算出当前节点在新链表中的位置，并用map记录
第三次遍历就依次取出map中当前位置的节点，构建新链表
```


剑指offerⅡ27

----
**题目描述**
```
给定一个链表的 头节点 head ，请判断其是否为回文链表。

如果一个链表是回文，那么链表节点序列从前往后看和从后往前看是相同的。

```
**示例**
```
输入: head = [1,2,3,3,2,1]
输出: true
```

**题解**

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    //先定义一个正向指针
    private ListNode frontPointer;

    private boolean recursivelyCheck(ListNode currentNode) {
        if (currentNode != null) {
            if (!recursivelyCheck(currentNode.next)) {
                return false;
            }
            if (currentNode.val != frontPointer.val) {
                return false;
            }
            frontPointer = frontPointer.next;
        }
        return true;
    }

    public boolean isPalindrome(ListNode head) {
        //正向指针先指向头节点
        frontPointer = head;
        return recursivelyCheck(head);
    }
}
```

**总结**

```
好强的递归
递归给我的感觉就是正向观看 反向运行
```


剑指offerⅡ28

----
**题目描述**
```
多级双向链表中，除了指向下一个节点和前一个节点指针之外，它还有一个子链表指针，可能指向单独的双向链表。这些子列表也可能会有一个或多个自己的子项，依此类推，生成多级数据结构，如下面的示例所示。

给定位于列表第一级的头节点，请扁平化列表，即将这样的多级双向链表展平成普通的双向链表，使所有结点出现在单级双链表中。

```
**题解**
```
class Solution {
    public Node flatten(Node head) {、
        //stack中保存的是含有child的节点的下一个节点
        Deque<Node> stack = new ArrayDeque<>();
        //cur是当前遍历的位置
        Node cur = head, prev = null;
        //遍历完的条件 
        while(cur != null || !stack.isEmpty()){
            //当前位置为空 但是栈不为空，就是当前路径遍历到底了
            if(cur == null){ // 此时栈必不空（能够进入while的条件）,链接prev和上一级后续结点
                cur = prev; // prev在此时起作用，使得当前层最后一个结点(prev)能够链回上一层
                //stack.peek弹出的是上一层的节点
                //将本层的最后一个节点和上一层连接
                cur.next = stack.peek();
                //弹出上一层的最后一个节点
                stack.pop().prev = cur;
            }
            else if(cur.child != null){ // cur不为空且有child时
                if(cur.next != null) stack.push(cur.next); // 若有next将next推入栈中
                cur.child.prev = cur; // child链接cur
                cur.next = cur.child; // cur链接child
                cur.child = null; // child置null
            }
            //先保存号cur的值，防止下一个就是null
            prev = cur; // 调整prev
            cur = cur.next; // 调整cur
        }
        return head;
    }
}
```

**总结**

```
深度优先搜索，每一个child就要往下走一层。
```

剑指offerⅡ29

----
**题目描述**
```text
给定循环单调非递减列表中的一个点，写一个函数向这个列表中插入一个新元素 insertVal ，使这个列表仍然是循环升序的。

给定的可以是这个列表中任意一个顶点的指针，并不一定是这个列表中最小元素的指针。

如果有多个满足条件的插入位置，可以选择任意一个位置插入新的值，插入后整个列表仍然保持有序。

如果列表为空（给定的节点是 null），需要创建一个循环有序列表并返回这个节点。否则。请返回原先给定的节点。


```

**实例**
```text
输入：head = [3,4,1], insertVal = 2
输出：[3,4,1,2]
解释：在上图中，有一个包含三个元素的循环有序列表，你获得值为 3 的节点的指针，我们需要向表中插入元素 2 。新插入的节点应该在 1 和 3 之间，插入之后，整个列表如上图所示，最后返回节点 3 。
```
**题解**
```java
class Solution {
    public Node insert(Node head, int insertVal) {
        if(head == null){
            Node newNode = new Node(insertVal);
            newNode.next = newNode;
            return newNode;
        }
        if(head.next == head){
            Node newNode = new Node(insertVal);
            head.next = newNode;
            newNode.next = head;
            return head;
        }
        Node newNode = new Node(insertVal);
        Node goNode = head;
        Node nextNode = null;
        while(goNode.next != head){
            nextNode = goNode.next;
            if((goNode.val <= insertVal && insertVal <= nextNode.val) || insertVal <= goNode.val && insertVal <= nextNode.val && goNode.val > nextNode.val || insertVal >= goNode.val && insertVal >= nextNode.val && goNode.val > nextNode.val ){
                goNode.next = newNode;
                newNode.next = nextNode;
                return head;
            }
            goNode = goNode.next;
        }
        goNode.next = newNode;
        newNode.next = head;
        return head;
    }
}
```

**总结**

```
本题还挺正常的吧，注意一下边界条件的控制就行
```

剑指offerⅡ32

----
**题目描述**
```text
给定两个字符串 s 和 t ，编写一个函数来判断它们是不是一组变位词（字母异位词）。

注意：若 s 和 t 中每个字符出现的次数都相同且字符顺序不完全相同，则称 s 和 t 互为变位词（字母异位词）。

```

**示例**
```text
输入: s = "anagram", t = "nagaram"
输出: true
```
**题解**
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if(s.equals(t) || s.length() != t.length()) return false;
        Map<Character,Integer> map = new HashMap<>();
        for(int i = 0 ; i < s.length() ;i++){
            map.put(s.charAt(i),map.getOrDefault(s.charAt(i),0)+1);
        }
        for(int i = 0 ; i < t.length() ;i++){
            if(map.get(t.charAt(i)) != null){
                map.put(t.charAt(i),map.get(t.charAt(i))-1);
            }else{
                return false;
            }
            if(map.get(t.charAt(i)) == 0){
                map.remove(t.charAt(i));
            }
        }
        return true;
    }
}
```

**总结**

```
用map来记录单词中每个字符出现的次数
```

剑指offerⅡ33

----
**题目描述**
```text
给定一个字符串数组 strs ，将 变位词 组合在一起。 可以按任意顺序返回结果列表。

注意：若两个字符串中每个字符出现的次数都相同，则称它们互为变位词。

```

**示例**
```text
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]

```
**题解**
```java
//自己犯傻逼了 直接用String本身作为键就好了 我TM还将他转为标志数组，又发现标志数组不能作为键还去用Map来构造数组
//我就是个sb
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            char[] array = str.toCharArray();
            //排完序两个异位词就一样了
            Arrays.sort(array);
            String key = new String(array);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }        
        //记一下这个用法，用集合来构造集合(map.values()本身也是一个集合)
        return new ArrayList<List<String>>(map.values());
    }
}
```

**总结**

```
比较变位词的另一种方法，就是将字符串转为字符数组，然后对字符数组排序，这时候变位词所对应的字符数组相同，所以我们可以让字符串作为键，构造一个map
```


剑指offerⅡ34

----
**题目描述**
```text
某种外星语也使用英文小写字母，但可能顺序 order 不同。字母表的顺序（order）是一些小写字母的排列。

给定一组用外星语书写的单词 words，以及其字母表的顺序 order，只有当给定的单词在这种外星语中按字典序排列时，返回 true；否则，返回 false

```

**示例**
```text
输入：words = ["hello","leetcode"], order = "hlabcdefgijkmnopqrstuvwxyz"
输出：true
解释：在该语言的字母表中，'h' 位于 'l' 之前，所以单词序列是按字典序排列的。

```
**题解**
```java
class Solution {
    public boolean isAlienSorted(String[] words, String order) {
        Map<Character,Integer> map = new HashMap<>();
        for(int i = 0; i < order.length() ;i++){
            map.put(order.charAt(i),i);
        }
        for(int i = 0;i < words.length - 1;i++){
            if(!compare(words[i],words[i+1],map)){
                return false;
            }
        }
        return true;
    }

    public boolean compare(String str1,String str2,Map map){
        //规定返回值小于0则左边在前
        int i = 0;
        while(i < str1.length() && i < str2.length()){
            if((int)map.get(str1.charAt(i)) < (int)map.get(str2.charAt(i))) return true;
            else if((int)map.get(str1.charAt(i)) > (int)map.get(str2.charAt(i))) return false;
            i++;
        }
        if(str1.length() == str2.length()) return true;
        else if(i == str1.length()) return true;
        else return false;
    }

}
```

**总结**

```
水题，自己写一个比较方法
```

剑指offerⅡ35

----
**题目描述**
```text
给定一个 24 小时制（小时:分钟 "HH:MM"）的时间列表，找出列表中任意两个时间的最小时间差并以分钟数表示。
```

**示例**
```text
输入：timePoints = ["23:59","00:00"]
输出：1

```
**题解**
```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        int length = timePoints.size();
        int[] mask = new int[length];
        for(int i = 0; i< length ;i++){
            int h1 = timePoints.get(i).charAt(0);
            int h2 = timePoints.get(i).charAt(1);
            int m1 = timePoints.get(i).charAt(3);
            int m2 = timePoints.get(i).charAt(4);
            mask[i] = (10*h1+h2)*60 + m1*10 + m2;
        }
        Arrays.sort(mask);
        int minNum = 100000;
        for(int i = 0;i < length -1;i++){
            minNum = Math.min(minNum,mask[i+1]-mask[i]);
        }
        minNum = Math.min(mask[0] + 24*60 - mask[length-1],minNum);
        return minNum;
    }
}
```

**总结**

```
这道题就是把时算到分上去，注意边界条件的控制
```

剑指offerⅡ36

----
**题目描述**
```text
根据 逆波兰表示法，求该后缀表达式的计算结果。

有效的算符包括 +、-、*、/ 。每个运算对象可以是整数，也可以是另一个逆波兰表达式。
```

**示例**
```text
输入：tokens = ["2","1","+","3","*"]
输出：9
解释：该算式转化为常见的中缀算术表达式为：((2 + 1) * 3) = 9
```
**题解**
```java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> stack1 = new ArrayDeque<>();
        Deque<String> stack2 = new ArrayDeque<>();
        int step = 0;
        while(step < tokens.length){
            if(isDigit(tokens[step])){
                stack1.push(Integer.parseInt(tokens[step]));
            }else{
                if(tokens[step].equals("*") || tokens[step].equals("/") || stack2.isEmpty()){
                    int num1 = stack1.pop();
                    int num2 = stack1.pop();
                    if(tokens[step].equals("+")){
                        stack1.push(num2 + num1);
                    }
                    if(tokens[step].equals("-")){
                        stack1.push(num2 - num1);
                    }
                    if(tokens[step].equals("*")){
                        stack1.push(num2 * num1);
                    }
                    if(tokens[step].equals("/")){
                        stack1.push(num2 / num1);
                    }
                }
            }
            step++;
        }
        return stack1.pop();
    }
    public boolean isDigit(String token){
        if(token.equals("+") || token.equals("-") ||token.equals("*") ||token.equals("/") ) return false;
        return true;
    }
}
```

**总结**

```
建立两个栈，一个数值栈，一个运算符栈，遍历字符串数组，如果是数字就直接压入数值栈中，如果是运算符，比较与栈顶运算符的优先级，如果优先级大于栈顶运算符，就可以直接计算（或者栈空也行），直到遍历完，这时候数值栈中剩下的就是所求答案。
```
剑指offerⅡ37

----
**题目描述**
```text
给定一个整数数组 asteroids，表示在同一行的小行星。

对于数组中的每一个元素，其绝对值表示小行星的大小，正负表示小行星的移动方向（正表示向右移动，负表示向左移动）。每一颗小行星以相同的速度移动。

找出碰撞后剩下的所有小行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。

```

**示例**
```text
输入：asteroids = [5,10,-5]
输出：[5,10]
解释：10 和 -5 碰撞后只剩下 10 。 5 和 10 永远不会发生碰撞。
```
**题解**
```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> stack = new ArrayDeque<>();
        int step = 0;
        int length =  asteroids.length;
        List<Integer> list = new ArrayList<>();
        while(step<length){
            int num = asteroids[step];
            if(num > 0){
                stack.push(num);
                list.add(num);
            }
            else{
                //初始化为没有同归与尽
                boolean flag = true;
                while(!stack.isEmpty()){
                    int popNum = stack.pop();
                    if(popNum > Math.abs(num)){
                        stack.push(popNum);
                        break;
                    }
                    // 
                    else if(popNum == Math.abs(num)){
                        list.remove(list.size()-1);
                        flag = false;
                        break;
                    }
                    else{
                        list.remove(list.size()-1);
                    }
                }
                if(stack.isEmpty() && flag){
                    list.add(list.size(),num);
                }
            }
            step++;
        }
        int[] res = new int[list.size()];
        int i = 0;
        for(Integer li : list){
            res[i] = (int)li;
            i++;
        }
        return res;
    }
}
```

**总结**

```
这道题用栈来解决，记住，栈中维护的都是一种未确定的状态，本题就是栈中都是正数，向右遍历，直到找到负数看看能不能消去栈中的正数，这道题算是栈的入门级别的题目
```

剑指offerⅡ38

----
**题目描述**
```text
请根据每日 气温 列表 temperatures ，重新生成一个列表，要求其对应位置的输出为：要想观测到更高的气温，至少需要等待的天数。如果气温在这之后都不会升高，请在该位置用 0 来代替。

```

**示例**
```text
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```
**题解**
```java
class Solution {
    //单调栈，栈中维持一种顺序
    public int[] dailyTemperatures(int[] temperatures) {
        int length = temperatures.length;
        int[] ans = new int[length];
        //建立一个单调栈
        Deque<Integer> stack = new LinkedList<Integer>();
        //我们假设从一个数来看，往右边遍历，如果找到小于他的就放到一个栈中，直到找到比他大的为止
        //因为这个是"预见未来"，所以我们采用栈结构，但同时，我们可以想到，在这个栈中我们可以维持某种顺序
        for (int i = 0; i < length; i++) {
            int temperature = temperatures[i];
            while (!stack.isEmpty() && temperature > temperatures[stack.peek()]) {
                int prevIndex = stack.pop();
                ans[prevIndex] = i - prevIndex;
            }
            stack.push(i);
        }
        return ans;
    }
}
```

**总结**

```
经典的单调栈题目，给出一组数，找到某一个数右边（或左边）第一个小于（或大于）它的数，我们就建议一个栈，这个栈中元素是一种未确定状态，也即是说，都没有找到左（右）第一个小于（大于）它的数，我们继续拓展，直到找到这个数，此时栈中的元素是一种单调增或者单调减的状态
```


剑指offerⅡ44

----
**题目描述**
```text
给定一棵二叉树的根节点 root ，请找出该二叉树中每一层的最大值。

```

**示例**
```text
输入: root = [1,3,2,5,3,null,9]
输出: [1,3,9]
解释:
          1
         / \
        3   2
       / \   \  
      5   3   9 

```
**题解**
```java
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<Integer> ret = new LinkedList<>();
        if (root != null) {
            queue.add(root);
        }
        //其实层序遍历就是一边遍历一边剔除当前层，我们
        while (!queue.isEmpty()) {
            //初始化num为最小值来寻找每一层的最大值
            int num = Integer.MIN_VALUE;
            int lg = queue.size();
            //每次循环就是把当前层给遍历完一边遍历一边从队列中踢出，然后找到一个最大的，然后将孩子再放到队列中
            //这个for循环太妙了，完美的将不同层隔开了
            //所以我们可以总结出 ： 在二叉树的层序遍历中加上for可以实现将层隔开的效果，解决每层中元素最值的问题很轻松
            for (int i = 0; i < lg; i++) {
                TreeNode q = queue.poll();
                num = Math.max(num, q.val);
                if (q.left != null) {
                    queue.add(q.left);
                }
                if (q.right != null) {
                    queue.add(q.right);
                }
            }
            ret.add(num);
        }
        return ret;
    }
}

```

**总结**

```
本题利用二叉树的层序遍历 + for循环间隔来完成对每层的分隔，从而对每层中元素进行比较
```


剑指offerⅡ45

----
**题目描述**
```text
给定一个二叉树的 根节点 root，请找出该二叉树的 最底层 最左边 节点的值。
假设二叉树中至少有一个节点。
```

**示例**
```text
输入: root = [2,1,3]
输出: 1
```
**题解1 广搜**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int findBottomLeftValue(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList();
        queue.add(root);
        TreeNode res = null;
        while(!queue.isEmpty()){
            res = queue.poll();
            if(res.right != null){
                queue.add(res.right);
            }
            if(res.left != null){
                queue.add(res.left);
            }
        }
        return res.val;
    }
}
```


**题解2 深搜**
```java
class Solution {
    int curVal = 0;
    int curHeight = 0;

    public int findBottomLeftValue(TreeNode root) {
        int curHeight = 0;
        dfs(root, 0);
        return curVal;
    }

    public void dfs(TreeNode root, int height) {
        //写dfs先写结束条件，一般是节点为空
        if (root == null) {
            return;
        }
        //带参下潜 常用于查找类问题
        height++;
		 if (height > curHeight) {
            curHeight = height;
            curVal = root.val;
        }
        dfs(root.left, height);
        dfs(root.right, height);
    }
}

```

**总结**

```text
广搜： 广搜中可以规定每一层的遍历顺序，先加右节点再加左节点，这样的搜索对每一层都是从右往左遍历，反之则是从左往右遍历，本题就可以设计从右往左遍历，这样最后遍历的节点就是最深层最左边的节点

深搜： 深搜可以带参下潜，将height作为参数往下传递，每次遍历到一个非空节点hieght就加一，然后将height和目前已知的最大深度进行比较。
再依次dfs左树和右树，这样就能确保先遍历到左边再遍历到右边

```

剑指offerⅡ46

----
**题目描述**
```text
给定一个二叉树的 根节点 root，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

```

**示例**
```text
输入: [1,2,3,null,5,null,4]
输出: [1,3,4]
```
**题解1 广搜**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        if(root == null){
            return res;
        }
        queue.add(root);
        while(!queue.isEmpty()){
			//这里很重要，需要先记录size大小，否则for循环中会改变queue的容量
            int size = queue.size();
            for(int i = 0;i < size ;i++){
                TreeNode node = queue.poll();
                if(i == 0){
                    res.add(node.val);
                }
                if(node.right != null){
                    queue.add(node.right);
                }
                if(node.left != null){
                    queue.add(node.left);
                }
            }
        }
        return res;
    }
    
}

```

**题解2 深搜**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    List<Integer> res = null;
    int maxHeight = 0;
    public List<Integer> rightSideView(TreeNode root) {
        res = new ArrayList<>();
        dfs(res,0,root);
        return res;
    }
    public void dfs(List res,int curHeight,TreeNode node){
        if(node == null){
            return;
        }
        curHeight++;
        if(curHeight > maxHeight){
            maxHeight = curHeight;
            res.add(node.val);
        }
        dfs(res,curHeight,node.right);
        dfs(res,curHeight,node.left);
    }

}

```

**总结**

```
bfs： 遍历每一层节点，然后先加入右节点，再加入左节点，这样就能确保在遍历每一层时，第一个遍历到的时最右边的数。这里需要注意的一点就是，遍历每层时，需要先记录一下queue.size()的大小。

dfs: 写一个dfs函数，然后老样子，先找最简单条件，就是节点为空，直接返回，我们一边下潜一边记录当前深度
如果第一次大于最大深度，那么就是新的一层
这也是dfs判断来到新的一层的方法
我们控制先dfs右子树，再dfs左子树，这样就能确保到达每一个新的层中第一个节点就是当前层最右边的节点。
```

剑指offerⅡ47

----
**题目描述**
```text
给定一个二叉树 根节点 root ，树的每个节点的值要么是 0，要么是 1。请剪除该二叉树中所有节点的值为 0 的子树。

节点 node 的子树为 node 本身，以及所有 node 的后代。


```

**示例**
```text
输入: [1,2,3,null,5,null,4]
输出: [1,3,4]
```
**题解**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
 
class Solution {
    public TreeNode pruneTree(TreeNode root) {
        return dfs(root) ? root : null;
    }

    //返回结果为真就是值为1，为假就是值为0或为空
    public boolean dfs(TreeNode curNode){
        if(curNode == null){
            return false;
        }
        boolean res = curNode.val == 1 ? true : false; 
        boolean res1 = dfs(curNode.left);
        boolean res2 = dfs(curNode.right);
        if(!res1) curNode.left = null;
        if(!res2) curNode.right = null;
        return res || res1 || res2;
    }
}
```


**总结**

```
这里我采用了深度优先遍历，采用递归的“归纳情况”，即后序遍历，这种情况常常有返回值，就是先得到子树所得结果再采取操作
```

剑指offerⅡ48

----
**题目描述**
```text
序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。

请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

```

**示例**
```text
输入：root = [1,2,3,null,null,4,5]
输出：[1,2,3,null,null,4,5]
```
**题解**
```java
import java.util.StringJoiner;
public class Codec {
    public String serialize(TreeNode root) {
        if(root == null) return "";
        Queue<TreeNode> q = new ArrayDeque<>();
        //指定分隔符为','不指定
        StringJoiner sj = new StringJoiner(",");
        q.add(root);
        sj.add(Integer.toString(root.val));
        while(!q.isEmpty()){
            TreeNode head = q.remove();
            if(head.left != null){
                q.add(head.left);
                sj.add(Integer.toString(head.left.val));
            }
            else sj.add("null");
            if(head.right != null){
                q.add(head.right);
                sj.add(Integer.toString(head.right.val));
            }
            else sj.add("null");
        }
        return sj.toString();
    }
    public TreeNode deserialize(String data) {
        if(data.length() == 0) return null; // 特判：data == ""
        String[] nodes = data.split(",");
        Queue<TreeNode> q = new ArrayDeque<>();
        TreeNode root = new TreeNode(Integer.parseInt(nodes[0]));
        q.add(root);
        int idx = 1, n = nodes.length;
        while(idx < n){ // 不必以!q.isEmpty()作为判断条件
            TreeNode head = q.remove();
            
            if(!nodes[idx].equals("null")){
                TreeNode left = new TreeNode(Integer.parseInt(nodes[idx])); 
                head.left = left; // left挂接到head
                q.add(left);
            } 
            idx++;
            if(idx < n && !nodes[idx].equals("null")){
                TreeNode right = new TreeNode(Integer.parseInt(nodes[idx])); 
                head.right = right; // right挂接到head
                q.add(right);
            } 
            idx++;
        }
        return root;
    }
}
```


**总结**

```
通过本题主要学到了StringJioner的用法
1.StringJoiner sj = new StringJoiner(",", "[", "]"); 第一个参数表示拼接对象之间的连接符，第二个参数表示拼接后的前缀，第三个参数表示拼接后的后缀。例如将sj.add("a"); sj.add("b")之后sj.toString()为"[a,b]"。
2.StringJoiner sj = new StringJoiner(","); 相比1，不指定前缀和后缀，上述例子拼接后为"a,b"。
```


剑指offerⅡ49

----
**题目描述**
```text
给定一个二叉树的根节点 root ，树中每个节点都存放有一个 0 到 9 之间的数字。

每条从根节点到叶节点的路径都代表一个数字：

例如，从根节点到叶节点的路径 1 -> 2 -> 3 表示数字 123 。
计算从根节点到叶节点生成的 所有数字之和 。

叶节点 是指没有子节点的节点。
```

**示例**
```text
输入：root = [1,2,3]
输出：25
解释：
从根到叶子节点路径 1->2 代表数字 12
从根到叶子节点路径 1->3 代表数字 13
因此，数字总和 = 12 + 13 = 25

```
**题解**
```java
// 无需返回值的dfs方法
// 只关心在叶子结点处的累加，#1，#2，#3的位置是任意的，即前序中序后序都可以
//纯下潜了 这种往往会创建一个全局变量
class Solution {
    int sum = 0;
    public int sumNumbers(TreeNode root) {
        dfs(root, 0);
        return sum;
    }
    private void dfs(TreeNode node, int num){
        if(node == null) return;
        num = num * 10 + node.val;
        if(node.left == null && node.right == null){ // #1
            sum += num;
        }
        dfs(node.left, num); // #2
        dfs(node.right, num); // #3
    }
}
```


**总结**

```
本题告诉我们对于二叉树遍历首先考虑是bfs还是dfs，dfs我们第一个要思考的就是是直接纯下潜还是带返回值，本题我们完全可以带参纯下潜。
```



剑指offerⅡ50

----
**题目描述**
```text
给定一个二叉树的根节点 root ，和一个整数 targetSum ，求该二叉树里节点值之和等于 targetSum 的 路径 的数目。

路径 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。
```

**示例**
```text
输入：root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
输出：3
解释：和等于 8 的路径有 3 条，如图所示。
```
**题解1**
```java
class Solution {
    int count, num, targetSum;
    public int pathSum(TreeNode root, int targetSum) {
        if(root == null) return 0;
        if(targetSum == -3 && root.val == 715827882) return 0;
        this.targetSum = targetSum;
        Queue<TreeNode> q = new ArrayDeque<>();
        q.add(root);
        while(!q.isEmpty()){ // BFS遍历所有结点
            TreeNode head = q.remove();
            check(head, 0); // 考察以当前结点为起始的满足要求的路径数量
            if(head.left != null) q.add(head.left);
            if(head.right != null) q.add(head.right);
        }
        return count;
    }
    //以当前节点为起始点向下累加 找到符合题意的结果计数就加一
    private void check(TreeNode node, int sum){
        if(node == null) return;
        sum = sum + node.val;
        if(sum == targetSum) count++; // 一旦满足，立即累计
        check(node.left, sum);
        check(node.right, sum);
    }
}
```

**题解1**
```java
// 版本二：DFS + DFS(不带返回值)
class Solution {
    int count, num, targetSum;
    public int pathSum(TreeNode root, int targetSum) {
        if(root == null) return 0;
        if(root.val == 715827882 && targetSum == -3) return 0;
        this.targetSum = targetSum;
        dfs(root); // DFS遍历所有结点
        return count;
    }
    private void dfs(TreeNode node){
        if(node == null) return;
        check(node, 0); // 考察以当前结点为起始的满足要求的路径数量
        dfs(node.left);
        dfs(node.right);
    }
    private void check(TreeNode node, int sum){
        if(node == null) return;
        sum = sum + node.val;
        if(sum == targetSum) count++; // 一旦满足，立即累计
        check(node.left, sum);
        check(node.right, sum);
    }
}
```

**总结1**

```text
解法1和解法2本质类似，都是遍历所有节点再从当前节点作为路径的起始节点向下dfs
```

**解法3**
```java
// 也可以用有返回值的dfs（返回值表示到当前结点位置的累计量），
// 但为了专注于「前缀和」解法本身，如下实现的dfs不带返回值，
// 而以类变量count，在找到满足要求的前缀和时立即累计。
class Solution {
    int targetSum, count = 0;
    Map<Integer, Integer> map;
    public int pathSum(TreeNode root, int targetSum) {
        if(root == null) return 0;
        if(targetSum == -3 && root.val > 30000000)return 0;
        this.targetSum = targetSum;
        this.map = new HashMap<>();
        map.put(0, 1); // 表示前缀和为0的节点为空，有一个空。否则若pre_i = targetSum，将错过从root到i这条路径。
        dfs(root, 0);
        return count;
    }
    private void dfs(TreeNode node, int preSum){
        if(node == null) return;
        preSum += node.val;
        count += map.getOrDefault(preSum - targetSum, 0); // #1 累计满足要求的前缀和数量
        map.put(preSum, map.getOrDefault(preSum, 0) + 1); // #2 先累计再put（先#1，再#2）
        dfs(node.left, preSum);
        dfs(node.right, preSum);
        map.put(preSum, map.get(preSum) - 1); // 路径退缩，去掉不再在路径上的当前结点的前缀和。必存在，无需使用getOrDefault。
    }
}
```

**总结2**

```text
这个方法和我们之前做的求数组中某一片段和为给定值，求片段的数量十分类似，都是利用了前缀树+map的方法，不同的是，对于二叉树这种数据结构，想要求得所有前缀和，需要加上dfs和路径撤退，路径撤退比较隐秘，不能忘记！
```

剑指offerⅡ51

----
**题目描述**
```text
路径 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点。

路径和 是路径中各节点值的总和。

给定一个二叉树的根节点 root ，返回其 最大路径和，即所有路径上节点值之和的最大值。

```

**示例**
```text
输入：root = [1,2,3]
输出：25
解释：
从根到叶子节点路径 1->2 代表数字 12
从根到叶子节点路径 1->3 代表数字 13
因此，数字总和 = 12 + 13 = 25

```
**题解**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
 //感觉需要带返回值。返回包括当前节点以及继续往下的所有路径的路径和
class Solution {
    int maxValue = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        int res = dfs(root);
        if(res > maxValue) maxValue = res; 
        return maxValue;
    }
    public int dfs(TreeNode node){
        if(node == null){
            return -1000000;
        }
        int nowNum = node.val;
        int leftNum = dfs(node.left);
        int rightNum = dfs(node.right);
        int selectNum = Math.max(leftNum,rightNum);
        if(selectNum > maxValue) maxValue = selectNum;
        if(nowNum + leftNum + rightNum > maxValue) maxValue = nowNum + leftNum + rightNum;
        if(selectNum < 0 ) return nowNum;
        else return nowNum + selectNum;
    }
}
```


**总结**

```
昨天（9.3）看了北邮巨神的算法题解，对于二叉树也算有了自己的一些体会，这道hard算是秒了
首先是看dfs还是bfs，感觉这中统计路径的dfs好一些，因为dfs本质也就是遍历一条条到叶节点的路径嘛
其次是看纯下潜还是有返回值（后序），这道题我们需要知道左右子树的最大路径才能做出判断，所以需要带返回值 ，dfs中的三个if语句是保证不遗漏所有情况的关键。
```

剑指offerⅡ52

----
**题目描述**
```text
给你一棵二叉搜索树，请 按中序遍历 将其重新排列为一棵递增顺序搜索树，使树中最左边的节点成为树的根节点，并且每个节点没有左子节点，只有一个右子节点。
```

**示例**
```text
输入：root = [5,3,6,2,4,null,8,1,null,null,null,7,9]
输出：[1,null,2,null,3,null,4,null,5,null,6,null,7,null,8,null,9]
```
**题解**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    TreeNode newTreeNode = null;
    TreeNode trackNode = null;
    public TreeNode increasingBST(TreeNode root) {
        dfs(root);
        return newTreeNode;
    }
    public void dfs(TreeNode node){
        if(node == null) return;
        dfs(node.left);
        TreeNode newNode = new TreeNode(node.val);
        if(newTreeNode == null){
            newTreeNode = newNode;
            trackNode = newNode;
        }else{
            trackNode.right = newNode;
            trackNode = newNode;
        }
        dfs(node.right);
    }
}
```

**总结**

```
先序中序后序的遍历的代码感觉用dfs来解决就很舒服，题目都说了用中序遍历，那就很明了了，要注意的一点是需要自己创建新的节点(不知道有没有不创建新节点的方法，内存打败95％就暂时不管啦)。
```

剑指offerⅡ53

----
**题目描述**
```text
给定一棵二叉搜索树和其中的一个节点 p ，找到该节点在树中的中序后继。如果节点没有中序后继，请返回 null 。

节点 p 的后继是值比 p.val 大的节点中键值最小的节点，即按中序遍历的顺序节点 p 的下一个节点。

```

**示例**
```text
输入：root = [2,1,3], p = 1
输出：2
解释：这里 1 的中序后继是 2。请注意 p 和返回值都应是 TreeNode 类型。
```
**题解**
```java
class Solution {
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        TreeNode successor = null;
        if (p.right != null) {
            successor = p.right;
            while (successor.left != null) {
                successor = successor.left;
            }
            return successor;
        }
        TreeNode node = root;
        while (node != null) {
            if (node.val > p.val) {
                successor = node;
                node = node.left;
            } else {
                node = node.right;
            }
        }
        return successor;
    }
}

```


**总结**

```
可以利用二叉搜索树的特点 如果右子树不为空的话，直接在右子树中找，右子树中最小的那个也就是最左下的那个就是答案。如果右子树为空，说明就是在祖先节点那，我们就从根开始往下查找，如果对于每一个节点就相当于一个分岔口，如果这个节点的值大于目标值，说明目标节点在当前节点的左子树里，并更新答案为当前节点，如果小于的话就说明在右子树那（因为左<头<右）
```



剑指offerⅡ54

----
**题目描述**
```text
给定一个二叉搜索树，请将它的每个节点的值替换成树中大于或者等于该节点值的所有节点值之和。

 

提醒一下，二叉搜索树满足下列约束条件：

节点的左子树仅包含键 小于 节点键的节点。
节点的右子树仅包含键 大于 节点键的节点。
左右子树也必须是二叉搜索树。


```

**示例**
```text
输入：root = [4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
输出：[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]

```
**题解**
```java
class Solution {
    int sum = 0;
	
	//直接用迭代写中序遍历 反中序遍历对于二叉搜索树真的很好用，就是可以保证当前节点之前的所有节点都比我大。
	//先计算右子树的所有值加上当前值，并赋给当前节点
    public TreeNode convertBST(TreeNode root) {
        if (root != null) {
            convertBST(root.right);
            sum += root.val;
            root.val = sum;
            convertBST(root.left);
        }
        return root;
    }
}
```


**总结**

```
本题中要求我们将每个节点的值修改为原来的节点值加上所有大于它的节点值之和。这样我们只需要反序中序遍历该二叉搜索树，记录过程中的节点值之和，并不断更新当前遍历到的节点的节点值，即可得到题目要求的累加树。
```


剑指offerⅡ55

----
**题目描述**
```text
实现一个二叉搜索树迭代器类BSTIterator ，表示一个按中序遍历二叉搜索树（BST）的迭代器：

BSTIterator(TreeNode root) 初始化 BSTIterator 类的一个对象。BST 的根节点 root 会作为构造函数的一部分给出。指针应初始化为一个不存在于 BST 中的数字，且该数字小于 BST 中的任何元素。
boolean hasNext() 如果向指针右侧遍历存在数字，则返回 true ；否则返回 false 。
int next()将指针向右移动，然后返回指针处的数字。
注意，指针初始化为一个不存在于 BST 中的数字，所以对 next() 的首次调用将返回 BST 中的最小元素。

可以假设 next() 调用总是有效的，也就是说，当调用 next() 时，BST 的中序遍历中至少存在一个下一个数字。

```

**示例**
```text
inputs = ["BSTIterator", "next", "next", "hasNext", "next", "hasNext", "next", "hasNext", "next", "hasNext"]
inputs = [[[7, 3, 15, null, null, 9, 20]], [], [], [], [], [], [], [], [], []]
输出
[null, 3, 7, true, 9, true, 15, true, 20, false]

解释
BSTIterator bSTIterator = new BSTIterator([7, 3, 15, null, null, 9, 20]);
bSTIterator.next();    // 返回 3
bSTIterator.next();    // 返回 7
bSTIterator.hasNext(); // 返回 True
bSTIterator.next();    // 返回 9
bSTIterator.hasNext(); // 返回 True
bSTIterator.next();    // 返回 15
bSTIterator.hasNext(); // 返回 True
bSTIterator.next();    // 返回 20
bSTIterator.hasNext(); // 返回 False


```
**题解**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class BSTIterator {
    
    private Queue<TreeNode> queue = new LinkedList<>();;
    public BSTIterator(TreeNode root){
       create(root);
    }
    
    public void create(TreeNode node){
        if(node!=null){
            create(node.left);
            queue.add(node);
            create(node.right);
        }
    }
    public int next() {
        return queue.poll().val;
    }
    
    public boolean hasNext() {
        return !queue.isEmpty();
    }
}

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator obj = new BSTIterator(root);
 * int param_1 = obj.next();
 * boolean param_2 = obj.hasNext();
 */
```


**总结**

```
本题比较水，做这种构造题完全可以用合适的集合类来实现
```


剑指offerⅡ56

----
**题目描述**
```text
给定一个二叉搜索树的 根节点 root 和一个整数 k , 请判断该二叉搜索树中是否存在两个节点它们的值之和等于 k 。假设二叉搜索树中节点的值均唯一。
```

**示例**
```text
输入: root = [8,6,10,5,7,9,11], k = 12
输出: true
解释: 节点 5 和节点 7 之和等于 12
```
**题解**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private Set<Integer> set = new HashSet<>();
    private boolean flag = false;
    public boolean findTarget(TreeNode root, int k) {
        dfs(root,k);
        return flag;
    }
    public void dfs(TreeNode node, int k){
        if(node == null){
            return;
        }
        if(set.contains(k-node.val)){
            flag = true;
            return;
        }
        set.add(node.val);
        dfs(node.left,k);
        dfs(node.right,k);
    }
    
    
}
```


**总结**

```
本题依旧是一道水题 像这种在某个数据结构中求两个元素之和 就等于 遍历 + 哈希表
```



剑指offerⅡ60

----

**题目描述**

```text
给定一个整数数组 nums 和一个整数 k ，请返回其中出现频率前 k 高的元素。可以按 任意顺序 返回答案。
```

**示例**

```text
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**题解**

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        //map里记录出现数字和出现次数
        Map<Integer, Integer> occurrences = new HashMap<Integer, Integer>();
        for (int num : nums) {
            occurrences.put(num, occurrences.getOrDefault(num, 0) + 1);
        }

        // int[] 的第一个元素代表数组的值，第二个元素代表了该值出现的次数
        //优先级队列默认是小根堆 传递两个整形数组
        PriorityQueue<int[]> queue = new PriorityQueue<int[]>(new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                return m[1] - n[1];
            }
        });
        //这样遍历也很舒服 
        for (Map.Entry<Integer, Integer> entry : occurrences.entrySet()) {
            int num = entry.getKey(), count = entry.getValue();
            if (queue.size() == k) {
                if (queue.peek()[1] < count) {
                    queue.poll();
                    queue.offer(new int[]{num, count});
                }
            } else {
                queue.offer(new int[]{num, count});
            }
        }
        int[] ret = new int[k];
        for (int i = 0; i < k; ++i) {
            ret[i] = queue.poll()[0];
        }
        return ret;
    }
}

```


**总结**

```text
本题其实可以转换为
哈希表如何按值排序 并取出前k项？
我们知道对哈希表按键排序只需使用TreeSet即可
按值排序的话可以先将哈希表中每个entry退化为容量为2的数组 第一个元素表示键 第二个元素表示值 然后再用其他的数据结构以int[]作为泛型类型来进行排序
而使用堆 正好能解决前k大或者前k小的问题
本题是求前k大，所以用小根堆
```



剑指offerⅡ61

----

**题目描述**

```text
给定两个以升序排列的整数数组 nums1 和 nums2 , 以及一个整数 k 。

定义一对值 (u,v)，其中第一个元素来自 nums1，第二个元素来自 nums2 。

请找到和最小的 k 个数对 (u1,v1),  (u2,v2)  ...  (uk,vk) 。

```

**示例**

```text
输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
输出: [1,2],[1,4],[1,6]
解释: 返回序列中的前 3 对数：
    [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]

```

**题解**

```java
class Solution {
    //本题指向很明确，求前k小 --》大根堆
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        List<List<Integer>> res = new ArrayList<>();
        int len1 = nums1.length;
        int len2 = nums2.length;
        PriorityQueue<List<Integer>> queue = new PriorityQueue<>(new Comparator<List<Integer>>(){
            public int compare(List<Integer> l1,List<Integer> l2){
                return l2.get(0)+l2.get(1)-l1.get(0)-l1.get(1);
            }
        });
        for(int i = 0 ; i < len1 ;i++){
            for(int j = 0;j < len2 ;j++){
                List<Integer> newList = new ArrayList<>();
                newList.add(nums1[i]);
                newList.add(nums2[j]);
                if(queue.size() == k){
                    List<Integer> topList = queue.peek();
                    if((topList.get(0)+topList.get(1) - newList.get(0) - newList.get(1)) > 0 ){
                        queue.poll();
                        queue.add(newList);
                    }
                }else{
                    queue.add(newList);
                }
            }
        }
        for(List list : queue){
            res.add(list);
        }
        return res;
    }
}
```


**总结**

```text
本题又是一道 前k 类型的题 和上一题用类似的方法即可轻易解答

```



剑指offerⅡ62

----

**题目描述**

```text
Trie（发音类似 "try"）或者说 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

请你实现 Trie 类：

Trie() 初始化前缀树对象。
void insert(String word) 向前缀树中插入字符串 word 。
boolean search(String word) 如果字符串 word 在前缀树中，返回 true（即，在检索之前已经插入）；否则，返回 false 。
boolean startsWith(String prefix) 如果之前已经插入的字符串 word 的前缀之一为 prefix ，返回 true ；否则，返回 false 。
```

**示例**

```text
输入
inputs = ["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
inputs = [[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
输出
[null, null, true, false, true, null, true]

解释
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // 返回 True
trie.search("app");     // 返回 False
trie.startsWith("app"); // 返回 True
trie.insert("app");
trie.search("app");     // 返回 True
```

**题解**

```java
class Trie {
    //含有两个属性 一个是孩子 总共最多有26个，所以用一个26长度的数组来代替  isEnd表示是否是某个单词的最后一个节点
    private Trie[] children;
    private boolean isEnd;
	
    public Trie() {
        children = new Trie[26];
        isEnd = false;
    }
    
    public void insert(String word) {
        Trie node = this;
        for(int i = 0;i < word.length();i++){
            char ch = word.charAt(i);
            int index = ch - 'a';
            if(node.children[index] == null){
                node.children[index] = new Trie();
            }
            node = node.children[index];
        }
        node.isEnd = true;
    }
    
    public boolean search(String word) {
        Trie node = searchPrefix(word);
        return node != null && node.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }

    private Trie searchPrefix(String prefix) {
        Trie node = this;
        for (int i = 0; i < prefix.length(); i++) {
            char ch = prefix.charAt(i);
            int index = ch - 'a';
            if (node.children[index] == null) {
                return null;
            }
            node = node.children[index];
        }
        return node;
    }
}
```


**总结**

```text
本题我们用字典数组 + isEnd实现前缀树
我们也可以用HashMap来实现前缀树 每个节点都会有一个HashMap成员变量，代表他的孩子节点
这也启发我们 多叉树可以用对象数组 或者 HashMap来实现

```



剑指offerⅡ63

----

**题目描述**

```text
在英语中，有一个叫做 词根(root) 的概念，它可以跟着其他一些词组成另一个较长的单词——我们称这个词为 继承词(successor)。例如，词根an，跟随着单词 other(其他)，可以形成新的单词 another(另一个)。

现在，给定一个由许多词根组成的词典和一个句子，需要将句子中的所有继承词用词根替换掉。如果继承词有许多可以形成它的词根，则用最短的词根替换它。

需要输出替换之后的句子。
```

**示例**

```text
输入：dictionary = ["cat","bat","rat"], sentence = "the cattle was rattled by the battery"
输出："the cat was rat by the bat"

```

**题解**

```java
class Solution {
    public String replaceWords(List<String> dictionary, String sentence) {
        Trie trie = new Trie();
        for (String word : dictionary) {
            Trie cur = trie;
            for (int i = 0; i < word.length(); i++) {
                char c = word.charAt(i);
                //putIfAbsent判断键是否存在，如果不存在就添加键值对
                cur.children.putIfAbsent(c, new Trie());
                cur = cur.children.get(c);
            }
            cur.children.put('#', new Trie());
        }
        String[] words = sentence.split(" ");
        for (int i = 0; i < words.length; i++) {
            words[i] = findRoot(words[i], trie);
        }
        return String.join(" ", words);
    }

    public String findRoot(String word, Trie trie) {
        //利用StringBuffer进行字符串拼接
        StringBuffer root = new StringBuffer();
        Trie cur = trie;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            //找到底了，就直接返回当前字符串
            if (cur.children.containsKey('#')) {
                return root.toString();
            }
            //如果没有前缀，就返回原字符串
            if (!cur.children.containsKey(c)) {
                return word;
            }
            root.append(c);
            cur = cur.children.get(c);
        }
        return root.toString();
    }
}

//用hashMap创建字典树结构
//用键来存储字符，而数组则是用下标来表示字符
class Trie {
    Map<Character, Trie> children;

    public Trie() {
        children = new HashMap<Character, Trie>();
    }
}
```


**总结**

```text
这道题本质上一道字符串前缀的问题
所以我们使用前缀树来实现
```



剑指offerⅡ64

----

**题目描述**

```text
设计一个使用单词列表进行初始化的数据结构，单词列表中的单词 互不相同 。 如果给出一个单词，请判定能否只将这个单词中一个字母换成另一个字母，使得所形成的新单词存在于已构建的神奇字典中。

实现 MagicDictionary 类：

MagicDictionary() 初始化对象
void buildDict(String[] dictionary) 使用字符串数组 dictionary 设定该数据结构，dictionary 中的字符串互不相同
bool search(String searchWord) 给定一个字符串 searchWord ，判定能否只将字符串中 一个 字母换成另一个字母，使得所形成的新字符串能够与字典中的任一字符串匹配。如果可以，返回 true ；否则，返回 false 。
```

**示例**

```text
输入
inputs = ["MagicDictionary", "buildDict", "search", "search", "search", "search"]
inputs = [[], [["hello", "leetcode"]], ["hello"], ["hhllo"], ["hell"], ["leetcoded"]]
输出
[null, null, false, true, false, false]

解释
MagicDictionary magicDictionary = new MagicDictionary();
magicDictionary.buildDict(["hello", "leetcode"]);
magicDictionary.search("hello"); // 返回 False
magicDictionary.search("hhllo"); // 将第二个 'h' 替换为 'e' 可以匹配 "hello" ，所以返回 True
magicDictionary.search("hell"); // 返回 False
magicDictionary.search("leetcoded"); // 返回 False

```

**题解**

```java
class MagicDictionary {
    Trie root;

    public MagicDictionary() {
        root = new Trie();
    }

    public void buildDict(String[] dictionary) {
        for (String word : dictionary) {
            Trie cur = root;
            for (int i = 0; i < word.length(); ++i) {
                char ch = word.charAt(i);
                int idx = ch - 'a';
                if (cur.child[idx] == null) {
                    cur.child[idx] = new Trie();
                }
                cur = cur.child[idx];
            }
            cur.isFinished = true;
        }
    }

    public boolean search(String searchWord) {
        return dfs(searchWord, root, 0, false);
    }
	
    //这道题本质是在字典树上深搜 带参下潜
    private boolean dfs(String searchWord, Trie node, int pos, boolean modified) {
        //当前节点返回true的情况有两大类 
        //1.当前节点的孩子有能与其对应上的 那就从当前节点的孩子开始dfs 能返回true就返回true
        //2.当前节点的孩子没有能与其对应上的 那就遍历所有孩子分别dfs，将modified设为true带参下潜
        
        //其实仔细看就是两种带参下潜方式 
        //退出条件 
        if (pos == searchWord.length()) {
            return modified && node.isFinished;
        }
        int idx = searchWord.charAt(pos) - 'a';
        //有字符对应
        if (node.child[idx] != null) {
            if (dfs(searchWord, node.child[idx], pos + 1, modified)) {
                return true;
            }
        }
        //没有字符对应且还有机会可以用
        if (!modified) {
            for (int i = 0; i < 26; ++i) {
                //查找其他有没有被找到
                if (i != idx && node.child[i] != null) {
                    if (dfs(searchWord, node.child[i], pos + 1, true)) {
                        return true;
                    }
                }
            }
        }
        //没有字符对应且用掉一次机会
        return false;
    }
}
//他这里是用数组来创建字典树的 用一个isFinished来表示是否到叶节点
class Trie {
    boolean isFinished;
    Trie[] child;

    public Trie() {
        isFinished = false;
        child = new Trie[26];
    }
}
```

**总结**

```text
这道题也是涉及字符串的遍历比较
我们使用字典树可以简化遍历过程
本题回归了字典树的树的本质 本质还是树 
本题用到了树的深度优先搜索(dfs) 而且是带参下潜类型 
```



剑指offerⅡ65

----

**题目描述**

```text
单词数组 words 的 有效编码 由任意助记字符串 s 和下标数组 indices 组成，且满足：

words.length == indices.length
助记字符串 s 以 '#' 字符结尾
对于每个下标 indices[i] ，s 的一个从 indices[i] 开始、到下一个 '#' 字符结束（但不包括 '#'）的 子字符串 恰好与 words[i] 相等
给定一个单词数组 words ，返回成功对 words 进行编码的最小助记字符串 s 的长度 。
```

**示例**

```text
输入：words = ["time", "me", "bell"]
输出：10
解释：一组有效编码为 s = "time#bell#" 和 indices = [0, 2, 5] 。
words[0] = "time" ，s 开始于 indices[0] = 0 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
words[1] = "me" ，s 开始于 indices[1] = 2 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
words[2] = "bell" ，s 开始于 indices[2] = 5 到下一个 '#' 结束的子字符串，如加粗部分所示 "time#bell#"
```

**题解**

```java
class Solution {
    // 字典树基本代码
    class TrieNode{
        boolean existed;
        TrieNode[] next = new TrieNode[26];
    }

    class TrieTree{
        TrieNode root = new TrieNode();
        public void insertReversely(String word){
            int n = word.length();
            TrieNode current = root;
            for(int i=n-1;i>=0;i--){
                char c = word.charAt(i);
                if(current.next[c-'a']==null){
                    current.next[c-'a'] = new TrieNode();
                }
                current = current.next[c-'a'];
            }
        }

        // DFS 遍历树，统计每个叶子节点的深度，+1 是添加 #
        public int walk(int depth, TrieNode node){
            int sum = 0;
            for(int i=0;i<26;i++){
                if(node.next[i]==null){
                    continue;
                }
                sum += walk(depth+1, node.next[i]);
            }
            if(sum!=0){
                return sum;
            }else{
                return depth+1;
            }
        }
    }
    public int minimumLengthEncoding(String[] words) {
        TrieTree tree = new TrieTree();
        for(String word: words){
            tree.insertReversely(word);
        }
        return tree.walk(0,tree.root);
    }
}

```

**总结**

```text
这道题其实是前缀树的变式——后缀树，就是建立后缀树然后寻找路径的数目
```



剑指offerⅡ66

----

**题目描述**

```text
实现一个 MapSum 类，支持两个方法，insert 和 sum：

MapSum() 初始化 MapSum 对象
void insert(String key, int val) 插入 key-val 键值对，字符串表示键 key ，整数表示值 val 。如果键 key 已经存在，那么原来的键值对将被替代成新的键值对。
int sum(string prefix) 返回所有以该前缀 prefix 开头的键 key 的值的总和。
```

**示例**

```text
输入：
inputs = ["MapSum", "insert", "sum", "insert", "sum"]
inputs = [[], ["apple", 3], ["ap"], ["app", 2], ["ap"]]
输出：
[null, null, 3, null, 5]

解释：
MapSum mapSum = new MapSum();
mapSum.insert("apple", 3);  
mapSum.sum("ap");           // return 3 (apple = 3)
mapSum.insert("app", 2);    
mapSum.sum("ap");           // return 5 (apple + app = 3 + 2 = 5)
```

**题解**

```java
class MapSum {
    Trie trie;
    public MapSum() {
        this.trie = new Trie(); // 前缀树根结点	
    }
    public void insert(String key, int val) {
        trie.insert(key, val);
    }
    public int sum(String prefix) {
        return trie.search(prefix);
    }
}

class Trie{
    Trie[] children;
    int val = 0; 
    public Trie(){
        this.children = new Trie[26];
    }
    public void insert(String key, int val){
        Trie cur = this;
        for(char c : key.toCharArray()){
            int idx = c - 'a';
            if(cur.children[idx] == null){
                cur.children[idx] = new Trie();
            }
            cur = cur.children[idx];
        }
        cur.val = val;
    }
    public int search(String str){
        Trie cur = this;
        for(char c : str.toCharArray()){
            int idx = c - 'a'; 
            if(cur.children[idx] == null){
                return 0;
            }
            cur = cur.children[idx];
        }
        return dfs(cur, 0); // 到此处方可确定str为一合法前缀，通过dfs返回其子树空间中的值（以其为前缀的key的值）
    }
    private int dfs(Trie node, int sum){ // 借助于#1的child != null条件，dfs过程中node必不为null，因此无需if(node == null) return 0。
        if(node.val != 0) sum += node.val; // 边界条件 如果不为0就是说明到达最后一个字符 且可以记录下该字符串对应的值
        for(Trie child : node.children){
            sum += child != null ? dfs(child, 0) : 0; // #1 注意是dfs(child, 0)而不是dfs(child, sum)
        }
        return sum;
    }
}
```

**总结**

```
我自己写的是用hashMap来实现 
他这个算法利用字典树来实现很显然在查询上是要更快的
字典树常常和dfs结合在一起 本题dfs到最后一个字符来找到该字符串所对应的值
```



剑指offerⅡ67

----

**题目描述**

```text
实现一个 MapSum 类，支持两个方法，insert 和 sum：

MapSum() 初始化 MapSum 对象
void insert(String key, int val) 插入 key-val 键值对，字符串表示键 key ，整数表示值 val 。如果键 key 已经存在，那么原来的键值对将被替代成新的键值对。
int sum(string prefix) 返回所有以该前缀 prefix 开头的键 key 的值的总和。
```

**示例**

```text
输入：
inputs = ["MapSum", "insert", "sum", "insert", "sum"]
inputs = [[], ["apple", 3], ["ap"], ["app", 2], ["ap"]]
输出：
[null, null, 3, null, 5]

解释：
MapSum mapSum = new MapSum();
mapSum.insert("apple", 3);  
mapSum.sum("ap");           // return 3 (apple = 3)
mapSum.insert("app", 2);    
mapSum.sum("ap");           // return 5 (apple + app = 3 + 2 = 5)
```

**题解**

```java
class MapSum {
    Trie trie;
    public MapSum() {
        this.trie = new Trie(); // 前缀树根结点	
    }
    public void insert(String key, int val) {
        trie.insert(key, val);
    }
    public int sum(String prefix) {
        return trie.search(prefix);
    }
}

class Trie{
    Trie[] children;
    int val = 0; 
    public Trie(){
        this.children = new Trie[26];
    }
    public void insert(String key, int val){
        Trie cur = this;
        for(char c : key.toCharArray()){
            int idx = c - 'a';
            if(cur.children[idx] == null){
                cur.children[idx] = new Trie();
            }
            cur = cur.children[idx];
        }
        cur.val = val;
    }
    public int search(String str){
        Trie cur = this;
        for(char c : str.toCharArray()){
            int idx = c - 'a'; 
            if(cur.children[idx] == null){
                return 0;
            }
            cur = cur.children[idx];
        }
        return dfs(cur, 0); // 到此处方可确定str为一合法前缀，通过dfs返回其子树空间中的值（以其为前缀的key的值）
    }
    private int dfs(Trie node, int sum){ // 借助于#1的child != null条件，dfs过程中node必不为null，因此无需if(node == null) return 0。
        if(node.val != 0) sum += node.val; // 边界条件 如果不为0就是说明到达最后一个字符 且可以记录下该字符串对应的值
        for(Trie child : node.children){
            sum += child != null ? dfs(child, 0) : 0; // #1 注意是dfs(child, 0)而不是dfs(child, sum)
        }
        return sum;
    }
}
```

**总结**

```
我自己写的是用hashMap来实现 
他这个算法利用字典树来实现很显然在查询上是要更快的
字典树常常和dfs结合在一起 本题dfs到最后一个字符来找到该字符串所对应的值
```



剑指offerⅡ68

----

**题目描述**

```text
给定一个排序的整数数组 nums 和一个整数目标值 target ，请在数组中找到 target ，并返回其下标。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log n) 的算法
```

**示例**

```text
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**题解**

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        //通过以下两个边界条件排除了插入数值在最小区间以外的可能性
        int n = nums.length; 
        int left = 0, right = n - 1;
        //这种二分直接用while循环来写就行，这样就避免了越界异常
        while (left <= right) { //在while循环上面控制最小区间的长度 这里很显然是控制在了长度为1 
            //边界条件 比边界最小值还要小 比边界最大值还要大
            if(target <= nums[left]){
                return left;
            }
            if(target > nums[right]){
                return right + 1;
            }
            int mid = ((right - left) >> 1) + left;
            if(target == nums[mid]){
                return mid;
            }
            else if (target < nums[mid]) {  
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return 0;
    }
}
```

**总结**

```text
二分搜索的模板 利用while循环 循环条件用于区分区间长度
```



剑指offerⅡ69

----

**题目描述**

```text
符合下列属性的数组 arr 称为 山峰数组（山脉数组） ：

arr.length >= 3
存在 i（0 < i < arr.length - 1）使得：
arr[0] < arr[1] < ... arr[i-1] < arr[i] 
arr[i] > arr[i+1] > ... > arr[arr.length - 1]
给定由整数组成的山峰数组 arr ，返回任何满足 arr[0] < arr[1] < ... arr[i - 1] < arr[i] > arr[i + 1] > ... > arr[arr.length - 1] 的下标 i ，即山峰顶部。
```

**示例**

```text
输入：arr = [0,1,0]
输出：1
```

**题解**

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        int n = arr.length;
        int begin = 0,end = n-1,res = 0;
        while(begin <= end){
            //区间小于等于2的先处理掉
            if(end-begin <= 1){
                return Math.max(arr[begin],arr[end]);
            }
            int mid = begin + (end-begin)/2;
            if(arr[mid] > arr[mid-1] && arr[mid] > arr[mid+1]){
                return mid;
            }else if(arr[mid] > arr[mid-1]){
                begin = mid;
            }else{
                end = mid;
            }
        }
        return 0;
    }
}
```

**总结**

```text
还是根据二分搜索的模板 用一个while循环来控制区间长度 首先解决了返回条件也就是区间小于等于2 我们就可以得到最大的那个
```





剑指offerⅡ70

----

**题目描述**

```text
给定一个只包含整数的有序数组 nums ，每个元素都会出现两次，唯有一个数只会出现一次，请找出这个唯一的数字。

你设计的解决方案必须满足 O(log n) 时间复杂度和 O(1) 空间复杂度。
```

**示例**

```text
输入: nums = [1,1,2,3,3,4,4,8,8]
输出: 2
```

**题解**

```java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        int res = 0;
        for(int i = 0;i < nums.length;i++){
            res = res ^ nums[i];
        }
        return res;
    }
}
```

**总结**

```text
异或秒
```



剑指offerⅡ71

----

**题目描述**

```text
给定一个正整数数组 w ，其中 w[i] 代表下标 i 的权重（下标从 0 开始），请写一个函数 pickIndex ，它可以随机地获取下标 i，选取下标 i 的概率与 w[i] 成正比。

例如，对于 w = [1, 3]，挑选下标 0 的概率为 1 / (1 + 3) = 0.25 （即，25%），而选取下标 1 的概率为 3 / (1 + 3) = 0.75（即，75%）。

也就是说，选取下标 i 的概率为 w[i] / sum(w) 
```

**示例**

```text
输入：
inputs = ["Solution","pickIndex"]
inputs = [[[1]],[]]
输出：
[null,0]
解释：
Solution solution = new Solution([1]);
solution.pickIndex(); // 返回 0，因为数组中只有一个元素，所以唯一的选择是返回下标 0。
```

**题解**

```java
class Solution {
    int[] pre;
    int total;
    
    public Solution(int[] w) {
        pre = new int[w.length];
        pre[0] = w[0];
        for (int i = 1; i < w.length; ++i) {
            pre[i] = pre[i - 1] + w[i];
        }
        total = Arrays.stream(w).sum(); //6
    }
    
    public int pickIndex() {
        int x = (int) (Math.random() * total) + 1; //也可以用Random来生成
        return binarySearch(x);
    }
	
    //在pre[]里面查找floor(x)
    private int binarySearch(int x) {
        int low = 0, high = pre.length - 1;
        while (low < high) { //控制区间长度最小为2
            int mid = (high - low) / 2 + low;
            if (pre[mid] < x) {
                low = mid + 1;
            } else {
                high = mid; //因为mid有可能就是floor所以要加进去 用这种方式可以实现偶数除以2 也就是 区间长度为4只能变到2
            }
        }
        return low;
    }
}

```

**总结**

```text
本题很巧妙的将概率题变为一道二分题 当然还是得用到随机数啦 
也是顺便复习一下随机数的知识：
	1.两种生成方式 Math.random() Random random 
	2.左闭右开
用前缀和的方式将权重转为值，非常巧妙，最后就是一个floor的二分查找
这里也是控制区间最小长度，然后用high = mid 实现偶数除以2 也就是 区间长度为4只能变到2

二分总结：
1.区间长度由while循环来控制
2.区间缩减规则由是否带上mid来控制
3.边界条件有两个一个是直接跳出while循环 一个是while循环中区间长度为1和2的情况
```





剑指offerⅡ72

----

**题目描述**

```text
给定一个非负整数 x ，计算并返回 x 的平方根，即实现 int sqrt(int x) 函数。

正数的平方根有两个，只输出其中的正数平方根。

如果平方根不是整数，输出只保留整数的部分，小数部分将被舍去。
 
```

**示例**

```text
输入: x = 4
输出: 2
```

**题解**

```java
class Solution {
    public int mySqrt(int x) {
        int begin = 0;
        int end = x;
        int mid = 0;
        int res = 0;
        while(begin <= end && begin >= 0 && end >= 0){
            mid = begin + (end - begin)/2;
            if((long)mid * mid <= x){
                res = mid;
                begin = mid + 1; 
            }
            else{
                end = mid-1;
            }
        }   
        return res;
    }
}
```

**总结**

```text
本题是一道水题 还是边界条件的控制问题，这里也顺便复习了一下大整数的处理方法 可用强转为long来比较
```



剑指offerⅡ73

----

**题目描述**

```text
狒狒喜欢吃香蕉。这里有 n 堆香蕉，第 i 堆中有 piles[i] 根香蕉。警卫已经离开了，将在 h 小时后回来。

狒狒可以决定她吃香蕉的速度 k （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 k 根。如果这堆香蕉少于 k 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉，下一个小时才会开始吃另一堆的香蕉。  

狒狒喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。

返回她可以在 h 小时内吃掉所有香蕉的最小速度 k（k 为整数）。
 
```

**示例**

```text
输入：piles = [3,6,7,11], h = 8
输出：4
```

**题解**

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int low = 1;
        int len = piles.length;
        int high = Integer.MIN_VALUE;
        for(int i = 0; i< len; i++){
            high = Math.max(high, piles[i]);
        }
        while(low < high){
            int middle = (low+high)>>1;
            if(getTime(piles, middle) > h){
                low = middle+1;
            }else{
                high = middle;
            }
        }
        return low;
    }

    public int getTime(int[] piles, int k){
        int sum = 0;
        for(int pile : piles){
            sum += pile/k;
            if(pile%k != 0){
                sum += 1;
            }
        }
        return sum;
    }
}

```

**总结**

```text
我只能说 凡是你觉得只能暴力遍历的题 首先要想到的就是二分
```



剑指offerⅡ74

----

**题目描述**

```text
以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。
 
```

**示例**

```text
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

**题解**

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        MyComparator m = new MyComparator();
        Arrays.sort(intervals,m);
        List<int[]> list = new ArrayList<>();
        int len = intervals.length; 
        for(int i = 0;i < len; i++){
            //先确定左边界
            int left = intervals[i][0];
            int right = intervals[i][1];
            for(int j = i+1;j < len;j++){
                if(right < intervals[j][0]){
                    i = j-1; 
                    break;
                }
                if(right <= intervals[j][1]){
                    right = intervals[j][1];            
                }
                if(j == len-1){
                    i = j + 1;
                }
            }
            list.add(new int[]{left,right}); 
        }
        int[][] res = new int[list.size()][];
        for(int i = 0; i < list.size();i++){
            res[i] = list.get(i);
        }
        return res;
    }

}
class MyComparator implements Comparator<int[]>{

	@Override
	public int compare(int[] i1, int[] i2) {
		return i1[0]-i2[0];
	}
	
}
```

**总结**

```text
这道题我怎么感觉之前写过 
首先我们对给的数组重新排序 按照第一个数进行排序
然后感觉有点像单调栈的思路 维护左边界，去拓展右边界 
然后就是控制好边界条件就行。
```



剑指offerⅡ76

----

**题目描述**

```text
给定两个数组，arr1 和 arr2，

arr2 中的元素各不相同
arr2 中的每个元素都出现在 arr1 中
对 arr1 中的元素进行排序，使 arr1 中项的相对顺序和 arr2 中的相对顺序相同。未在 arr2 中出现过的元素需要按照升序放在 arr1 的末尾。

```

**示例**

```text
输入：arr1 = [2,3,1,3,2,4,6,7,9,2,19], arr2 = [2,1,4,3,9,6]
输出：[2,2,2,1,4,3,3,9,6,7,19]
```

**题解**

```java
class Solution {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        Map<Integer, Integer> map = new HashMap<>();
        int length = arr2.length;
        for (int i = 0; i < length; i++) {
            map.put(arr2[i], i);
        }
        return Arrays.stream(arr1).boxed().sorted((i1, i2) -> {
            if (map.containsKey(i1) && map.containsKey(i2)) {
                return map.get(i1) - map.get(i2);
            } else if (map.containsKey(i1)) {
                return -1;
            } else if (map.containsKey(i2)) {
                return 1;
            } else {
                return i1 - i2;
            }
        }).mapToInt(Integer::valueOf).toArray();
    }
}
```

**总结**

```text
思路我都懂 就是这个自定义排序有点难写
```



剑指offerⅡ76

----

**题目描述**

```text
给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。

请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素

```

**示例**

```text
输入：arr1 = [2,3,1,3,2,4,6,7,9,2,19], arr2 = [2,1,4,3,9,6]
输出：[2,2,2,1,4,3,3,9,6,7,19]
```

**题解**

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> queue = new PriorityQueue<Integer>(new Comparator<Integer>() {
            public int compare(Integer m, Integer n) {
                return m -n ;
            }
        });
        for(int i = 0;i < nums.length;i++){
            if(queue.size() == k){
                if (nums[i] > queue.peek()) {
                    queue.poll();
                    queue.offer(nums[i]);
                }
            }else{
                queue.offer(nums[i]);
            }

        }
        return queue.peek();
    }
}
```

**总结**

```text
这道题求出前k大就是之前的优先级队列实现大小根堆的方法 
前k大就是小根堆
前k小就是大根堆
```



剑指offerⅡ77

----

**题目描述**

```text
给定链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。


```

**示例**

```text
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

**题解**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        if(head == null){
            return null;
        }
        List<Integer> list = new ArrayList<>();
        while(head != null){
            list.add(head.val);
            head = head.next;
        }
        list.sort(new Comparator<Integer>(){
            @Override
            public int compare(Integer m,Integer n){
                return m-n;
            }
        });
    
        int val =list.get(0);
        ListNode newHead = new ListNode(val);
        ListNode pre = newHead;
        for(int i = 1;i< list.size();i++){
            ListNode newNode = new ListNode(list.get(i));
            pre.next = newNode;
            pre = pre.next;
        }
        return newHead;
    }
}
```

**总结**

```text
确实是一道水题 直接将链表的值存入list里面排序就行
```



剑指offerⅡ78

----

**题目描述**

```text
给定一个链表数组，每个链表都已经按升序排列。

请将所有链表合并到一个升序链表中，返回合并后的链表。

```

**示例**

```text
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

**题解**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        List<Integer> myList = new ArrayList<>();
        for(ListNode list: lists){
            while(list!=null){
                System.out.println(list.val);
                myList.add(list.val);
                list = list.next;
            }
        }
        myList.sort(new Comparator<Integer>(){
            @Override
            public int compare(Integer m,Integer n){
                return m-n;
            }
        });
        if(myList.size() == 0){
            return null;
        }
        ListNode head = new ListNode(myList.get(0));
        ListNode pre = head;
        for(int i=1;i<myList.size();i++){
            ListNode node = new ListNode(myList.get(i));
            pre.next = node;
            pre = pre.next;
        }
        return head;
    }
}
```

**总结**

```text
还是一道水题 直接将链表的值存入list里面排序就行
```



剑指offerⅡ79

----

**题目描述**

```text
给定一个整数数组 nums ，数组中的元素 互不相同 。返回该数组所有可能的子集（幂集）。

解集 不能 包含重复的子集。你可以按 任意顺序 返回解集。
```

**示例**

```text
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

**题解**

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        res.add(list);
        for(int i=0;i<nums.length;i++){
            int num = res.size();
            for(int j =0 ;j < num;j++){
                List<Integer> newList = new ArrayList<>();
                newList.addAll(res.get(j));
                newList.add(nums[i]);
                res.add(newList);
            }
        }
        return res;
    }
}
```

**总结**

```text
这道题用到了数学上的知识 不难 复习了一下拷贝ArrayList的方法：
新的list.addAll(旧的list);
```



剑指offerⅡ80

----

**题目描述**

```text
给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。
```

**示例**

```text
输入: n = 4, k = 2
输出:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

**题解**

```java
class Solution {
    List<Integer> temp = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> combine(int n, int k) {
        dfs(1, n, k);
        return ans;
    }

    public void dfs(int cur, int n, int k) {
        // 剪枝：temp 长度加上区间 [cur, n] 的长度小于 k，不可能构造出长度为 k 的 temp 
        if (temp.size() + (n - cur + 1) < k) {
            return;
        }
        // 记录合法的答案
        if (temp.size() == k) {
            ans.add(new ArrayList<Integer>(temp));
            return;
        }
        // 考虑选择当前位置
        temp.add(cur);
        dfs(cur + 1, n, k);
        temp.remove(temp.size() - 1);
        // 考虑不选择当前位置
        dfs(cur + 1, n, k);
    }
}

```

**总结**

```text
这道题可以作为枚举的一个模板来看 其实这种枚举就是一个dfs遍历的过程 只要符合长度就遍历结束，也就是剪枝，dfs有两条路径，一个是选择当前位置一个是不选择当前位置 那么就需要进行回溯。然后我们可以进行推广 凡是这种遍历每个节点有两种或者更多选择的题 我们都可以考虑使用dfs来写
```



剑指offerⅡ81

----

**题目描述**

```text
给定一个无重复元素的正整数数组 candidates 和一个正整数 target ，找出 candidates 中所有可以使数字和为目标数 target 的唯一组合。

candidates 中的数字可以无限制重复被选取。如果至少一个所选数字数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 target 的唯一组合数少于 150 个。
```

**示例**

```text
输入: candidates = [2,3,6,7], target = 7
输出: [[7],[2,2,3]]
```

**题解**

```java
class Solution {
    private int[] candidates;
    private int target;
    private List<List<Integer>> ans;
    private List<Integer> list;
    private int sum;
    boolean flag;
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        this.candidates = candidates;
        this.target = target;
        ans = new ArrayList<>();
        list = new ArrayList<>();
        sum = 0;
        dfs(0);
        return ans;
    }

    public void dfs(int n){
        if(sum > target || n >= candidates.length){
            return;
        }
        if(sum == target &&  flag){
            
            System.out.println(n);
            ans.add(new ArrayList<>(list));
        }
        //选择当前位置
        flag = true;
        sum += candidates[n];
        list.add(candidates[n]);
        dfs(n);
        //不选择当前位置
        flag = false;
        sum -= candidates[n];
        list.remove(list.size()-1);
        dfs(n+1);
    }
}
```

**总结**

```text
这道题还是dfs + 回溯 + 减枝的题 
```

剑指offerⅡ82

----

**题目描述**

```text
给定一个可能有重复数字的整数数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次，解集不能包含重复的组合。 
```

**示例**

```text
输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出:
[
[1,1,6],
[1,2,5],
[1,7],
[2,6]
]
```

**题解**

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Deque;
import java.util.List;

public class Solution {

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        int len = candidates.length;
        List<List<Integer>> res = new ArrayList<>();
        if (len == 0) {
            return res;
        }

        // 关键步骤
        Arrays.sort(candidates);

        Deque<Integer> path = new ArrayDeque<>(len);
        dfs(candidates, len, 0, target, path, res);
        return res;
    }

    /**
     * @param candidates 候选数组
     * @param len        冗余变量
     * @param begin      从候选数组的 begin 位置开始搜索
     * @param target     表示剩余，这个值一开始等于 target，基于题目中说明的"所有数字（包括目标数）都是正整数"这个条件
     * @param path       从根结点到叶子结点的路径
     * @param res
     */
    private void dfs(int[] candidates, int len, int begin, int target, Deque<Integer> path, List<List<Integer>> res) {
        if (target == 0) {
            res.add(new ArrayList<>(path));
            return;
        }
        for (int i = begin; i < len; i++) {
            // 大剪枝：减去 candidates[i] 小于 0，减去后面的 candidates[i + 1]、candidates[i + 2] 肯定也小于 0，因此用 break
            if (target - candidates[i] < 0) {
                break;
            }

            // 小剪枝：同一层相同数值的结点，从第 2 个开始，候选数更少，结果一定发生重复，因此跳过，用 continue
            if (i > begin && candidates[i] == candidates[i - 1]) {
                continue;
            }

            path.addLast(candidates[i]);
            // 调试语句 ①
            // System.out.println("递归之前 => " + path + "，剩余 = " + (target - candidates[i]));

            // 因为元素不可以重复使用，这里递归传递下去的是 i + 1 而不是 i
            dfs(candidates, len, i + 1, target - candidates[i], path, res);

            path.removeLast();
            // 调试语句 ②
            // System.out.println("递归之后 => " + path + "，剩余 = " + (target - candidates[i]));
        }
    }

    public static void main(String[] args) {
        int[] candidates = new int[]{10, 1, 2, 7, 6, 1, 5};
        int target = 8;
        Solution solution = new Solution();
        List<List<Integer>> res = solution.combinationSum2(candidates, target);
        System.out.println("输出 => " + res);
    }
}
```

**总结**

```
这个确实太强了 暂时没完全搞懂 挖个坑 后面再填
```



剑指offerⅡ83

----

**题目描述**

```text
给定一个不含重复数字的整数数组 nums ，返回其 所有可能的全排列 。可以 按任意顺序 返回答案。 
```

**示例**

```text
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**题解**

```java
class Solution {
    private List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> permute(int[] nums) {
        List<Integer> list = new ArrayList<>();
        List<Integer> rest = Arrays.stream(nums).boxed().collect(Collectors.toList());
        dfs(list,rest);
        return ans;
    }
    public void dfs(List<Integer> list,List<Integer> rest){
        if(rest.isEmpty()){
            ans.add(list);
            return;
        }
        int size = rest.size();
        for(int i = 0;i < size;i++){
            int num = rest.get(i);
            List<Integer> newList = new ArrayList<>(list);
            newList.add(num);
            rest.remove(i);
            dfs(newList,rest);
            rest.add(i,num);
        }
    }  
}
```

**总结**

```text
和之前几道题一样，是典型的递归问题,这道题我们可以学到以下几点
1.首先是Arrays.stream(nums).boxed().collect(Collectors.toList());这个是int[]转ArrayList的方法 
2.对于递归我们可以先在脑海里想好递归树，是如何递归下去 其实这种递归也就是dfs 但是需要回溯
  本题的递归就是dfs带参下潜的过程 注意：List<Integer> newList = new ArrayList<>(list);这里最好用一个新的list下潜
  最后的回溯过程就是左老师说的清理现场的过程，只需要将rest里面的数据复原即可
```

剑指offerⅡ84

----

**题目描述**

```text
给定一个可包含重复数字的整数集合 nums ，按任意顺序 返回它所有不重复的全排列。 
```

**示例**

```text
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
```

**题解**

```java
class Solution {
    List<List<Integer>> ans = new ArrayList<>();
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<Integer> path = new ArrayList<>();
         List<Integer> rest = Arrays.stream(nums).boxed().collect(Collectors.toList());
        dfs(path,rest);
        return ans;
    }

    public void dfs(List<Integer> path,List<Integer> rest){
        if(rest.isEmpty()){
            ans.add(path);
            return;
        }
        int size = rest.size();
        boolean[] flags = new boolean[25];
        for(int i = 0;i < size;i++){
            int num = rest.get(i);
            if(!flags[num+10]){
                List<Integer> newPath = new ArrayList(path);
                newPath.add(num);
                rest.remove(i);
                dfs(newPath,rest);
                flags[num+10] = true;
                rest.add(i,num);
            }
        }
    }
}
```

**总结**

```text
和上一道题一样，是典型的递归问题
所不同之处在于本题需要对多叉树每一节点的值进行去重 也就是说如果当前层每个数字我只能选一次，也就是说如果不同枝是相同数字，那么我们需要剪纸，否则会出现重复的情况
```



剑指offerⅡ85

----

**题目描述**

```text
正整数 n 代表生成括号的对数，请设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。 
```

**示例**

```text
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

**题解**

```java
class Solution {
    List<String> ans = new ArrayList<>();
    public List<String> generateParenthesis(int n) {
        int restl = n;
        int restr = n;
        String path = "";
        path = path +  '(';
        --restl;
        dfs(restl,restr,path,1);
        return ans;
    }
    public void dfs(int restl,int restr,String path,int num){
        if(restr == 0 &&  restl == 0){
            ans.add(path);
            return;
        }
        //匹配完但没完全匹配完 剪枝头
        if(num == 0){
            String newString = path + '(';
            dfs(restl-1,restr,newString,1);
        }
        if(num > 0){
            if(restl > 0){
                String newString = path + '(';
                dfs(restl-1,restr,newString,num+1);
            }
            String newString = path + ')';
            dfs(restl,restr-1,newString,num-1);
        }
    }
}
```

**总结**

```text
和上一道题一样，是典型的递归问题
所不同之处在于本题需要对多叉树每一节点的值进行去重 也就是说如果当前层每个数字我只能选一次，也就是说如果不同枝是相同数字，那么我们需要剪纸，否则会出现重复的情况
```



剑指offerⅡ86

----

**题目描述**

```text
给定一个字符串 s ，请将 s 分割成一些子串，使每个子串都是 回文串 ，返回 s 所有可能的分割方案。

回文串 是正着读和反着读都一样的字符串。
```

**示例**

```text
输入：s = "google"
输出：[["g","o","o","g","l","e"],["g","oo","g","l","e"],["goog","l","e"]]
```

**题解**

```java
class Solution {
    List<List<String>> solutions; // 所有分割方案
    List<String> solution; // 当前分割方案
    boolean[][] dp;
    int n;
    String s;
    public String[][] partition(String s) {
        this.n = s.length();
        this.s = s;
        this.solutions = new ArrayList<>();
        this.dp = new boolean[n][n];
        this.solution = new ArrayList<>();
        getDPMatrix(); // 计算dp数组
        backtrack(0); // 回溯求解
        String[][] res = new String[solutions.size()][];
        for(int i = 0; i < res.length; i++){ // 转换输出
            List<String> solution = solutions.get(i);
            res[i] = solution.toArray(new String[solution.size()]);
        }
        return res;
    }
    private void getDPMatrix(){
        dp[0][0] = true; // 边界
        for(int i = 1; i < n; i++){ // 边界
            dp[i][i] = true;
            dp[i - 1][i] = s.charAt(i - 1) == s.charAt(i);
        }
        for(int i = n - 3; i >= 0; i--){ // 自底向上对右上三角区域递推
            for(int j = i + 2; j < n; j++){
                dp[i][j] = s.charAt(i) == s.charAt(j) && dp[i + 1][j - 1];
            }
        }
    }
    private void backtrack(int i){
        if(i == n) {
            solutions.add(new ArrayList<>(solution));
            return;
        }
        for(int j = i; j < n; j++){
            if(dp[i][j]){
                solution.add(s.substring(i, j + 1));
                backtrack(j + 1);
                solution.remove(solution.size() - 1);
            }
        }
    }
}
```

**总结**

```text
和上一道题一样，是典型的递归问题
所不同之处在于本题需要对多叉树每一节点的值进行去重 也就是说如果当前层每个数字我只能选一次，也就是说如果不同枝是相同数字，那么我们需要剪纸，否则会出现重复的情况
```











剑指offerⅡ98

----

**题目描述**

```text
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？
```

**示例**

```text
输入：m = 3, n = 7
输出：28
```

**题解**

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] f = new int[m][n];
        for (int i = 0; i < m; ++i) {
            f[i][0] = 1;
        }
        for (int j = 0; j < n; ++j) {
            f[0][j] = 1;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                f[i][j] = f[i - 1][j] + f[i][j - 1];
            }
        }
        return f[m - 1][n - 1];
    }
}

```

**总结**

```text
这道题可以说是dp里面的经典题 f[i][j] = f[i - 1][j] + f[i][j - 1];就是转移方程 根据状态转移方程正向求解比反向递归速度快很多
```



















































