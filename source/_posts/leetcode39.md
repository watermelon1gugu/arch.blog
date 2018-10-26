---
title: leetcode39-组合总和题解
date: 2018-08-08 01:21:29
tags: 	
	- leetcode
	- 39
---

# 题目

给定一个**无重复元素**的数组 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的数字可以无限制重复被选取。

<!--*more*-->

**说明：**

- 所有数字（包括 `target`）都是正整数。
- 解集不能包含重复的组合。 

**示例 1:**

```
输入: candidates = [2,3,6,7], target = 7,
所求解集为:
[
  [7],
  [2,2,3]
]
```

**示例 2:**

```
输入: candidates = [2,3,5], target = 8,
所求解集为:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

# 题解#

常见的dfs递归问题,将问题进一步分解为是否将当前index指向的数字加入组合中

```
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int> &candidates, int target) {
        vector<vector<int>> result;
        vector<int> book;
        combinate(result, book, candidates, candidates.size() - 1, target);
        return result;
    }

private:
    void combinate(vector<vector<int>> &result, vector<int> &book, vector<int> &candidates, int index, int target) {
        if (target == 0) {
            result.push_back(book);
            return;
        }
        if (target < 0) {
            return;
        }

        if (index >= 0) {
            book.push_back(candidates[index]);
            combinate(result, book, candidates, index, target - candidates[index]);
            book.pop_back();
            combinate(result, book, candidates, index - 1, target);
        }
    }
    vector<vector<int>> getCartesianProduct(vector<vector<int>> &a, vector<vector<int>> &b) {
        vector<vector<int>> result;
        for (int i = 0; i < a.size(); i++) {
            for (int k = 0; k < b.size(); k++) {
                vector<int> temp;
                temp.insert(temp.end(), a[i].begin(), a[i].end());
                temp.insert(temp.end(), b[k].begin(), b[k].end());
                result.push_back(temp);
            }
        }
        return result;
    }
};
```

