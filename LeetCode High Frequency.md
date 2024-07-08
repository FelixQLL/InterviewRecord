[3]无重复字符的最长子串 **[滑动窗口]**
   ```
   class Solution {
   public:
    int lengthOfLongestSubstring(string s) {
      int res = 0;
      int left = 0, right = 0;
      std::unordered_map<char, int> map;
      while (right < s.size()) {
        if (map.count(s[right]) != 0) {
            if (map[s[right]] >= left) left = map[s[right]] + 1;
        }
        map[s[right]] = right;
        res = std::max(res, right - left + 1);
        right++;
      }
      return res;
    }
   };
```

[25] K个一组反转链表
[206] 反转链表
[215] 数组中的第K个最大元素
[15] 三数之和
[146] LRU缓存机制
[200] 岛屿数量
[121] 买卖股票的最佳时机
[42] 接雨水
[160] 相交链表
[53] 最大子序和
[5] 最长回文子串
[46] 全排列
[23] 合并K个排序链表
[21] 合并两个有序链表
[102] 二叉树的层序遍历
[141] 环形链表
[20] 有效的括号
[88] 合并两个有序数组
[92] 反转链表2
[300] 最长上升子序列
[143] 重排链表
[19] 删除链表的倒数第N个节点
[124] 二叉树中的最大路径和
[94] 二叉树的中序遍历
[1143] 最长公共子序列
[704] 二分查找
[232] 用栈实现队列
[4] 寻找两个正序数组的中位数
[148] 排序链表
[236] 二叉树的最近公共祖先
[33] 搜索旋转排序数组
[199] 二叉树的右视图
