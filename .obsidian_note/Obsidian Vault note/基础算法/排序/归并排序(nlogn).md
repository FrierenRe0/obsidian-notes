**本质就是先拆在合的过程
## 1. 核心复杂度

时间复杂度：稳定 \(O(nlog n)\)

- 最好 / 最坏 / 平均全是 \(O(nlog n)\)，不受原始数组有序程度影响
    空间复杂度：\(O(n)\)（需要辅助数组存合并结果）
    稳定性：**稳定排序**（相等元素相对顺序不变）

## 2. 核心思想（分治）

1. **分**：把数组对半拆成左右两段，递归拆分直到区间只剩 1 个元素（天然有序）
2. **治（合并）**：将两个有序小数组，合并成一个大有序数组


### 这里就用模板题演式归并排序：
#### [排序数组](https://leetcode.cn/problems/sort-an-array/)
归并排序代码如下：
```c++
class Solution {
public: 
    void merge(vector<int>& nums,int l,int r,int mid) {
        vector<int> tmp;
        tmp.reserve(r - l + 1);
        int left = l,right = mid + 1;
        while(left <= mid && right <= r) {
            if(nums[left] <= nums[right]) tmp.push_back(nums[left++]);
            else tmp.push_back(nums[right++]);
        }
        while(left <= mid) tmp.push_back(nums[left++]);
        while(right <= r) tmp.push_back(nums[right++]);
        for(int i = l;i <= r;i++) {
            nums[i] = tmp[i - l];
        }
    }
    void Break(vector<int>& nums,int l,int r) {
        if(l == r) return;
        int mid = (l + r) >> 1;
        Break(nums,l,mid);
        Break(nums,mid + 1,r);
        merge(nums,l,r,mid);
    }
    vector<int> sortArray(vector<int>& nums) {
        int n = nums.size();
        Break(nums,0,n - 1);
        return nums;
    }
};
```