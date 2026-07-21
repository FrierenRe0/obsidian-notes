## 1. ordered_set是一个红黑树的封装容器；

```c++
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
using ordered_set = tree<pair<int,int>,null_type,less<>,rb_tree_tag,tree _order_statistics_update>;//
```

## 2. multiset , multimap 的对比
**std::multiset /std::multimap 的核心特点**
允许**多个等价 key 同时存在**，重复插入不会失败，全部存进去。
而普通 `set/map` 不允许重复，重复插入直接无效，和 pb_ds tree（集合版）一样。

## 3. ordered_set和set的对比
**你的 `tree<pair<int,int>>` 本质 ≈ 模拟 multiset**
原生 pb_ds 没有提供 `multiset` 版本的有序树，所以业界通用 trick：
给存储的元素多加一个**唯一后缀**（你这里自增 idx），让原本相同的数值变成完全不同的 pair，就能多次插入，实现「可存多个相同主值」的效果，行为等价于 multiset。

**4.问题：那我为什么还要用ordered_set，居然ordered_set和set貌似大差不差？**

**用这个问题作为引入了解**：
[3739. 统计主要元素子数组数目 II](https://leetcode.cn/problems/count-subarrays-with-majority-element-ii/)

常规写法：（set/multiset/multimap）
```c++
class Solution {
public:
    long long countMajoritySubarrays(vector<int>& nums, int target) {
        int idx = 0;
        set<pair<int,int>> st;
        st.insert({0, ++idx});
        long long ans = 0;
        int s = 0;
        for (int x : nums) {
            s += x == target ? 1 : -1;
            auto pos = st.lower_bound({s, -1});
            if (pos != st.begin()) {
                auto it = prev(pos);
                ans += distance(st.begin(), it) + 1;//就是以为这里在计算<key的长度时，时间复杂度时0(k)(同理multimap),所以这里用ordered_set(因为这里的底层逻辑红黑树（平衡二叉搜索树）时，在计算时时间复杂度时logn);
            }
            st.insert({s, ++idx});
        }
        return ans;
    }
};
```
ordered_set写法：
```c++
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
// 使用 pair<key, index> 支持重复 key
using ordered_set = tree<pair<int, int>, null_type, less<>, rb_tree_tag, tree_order_statistics_node_update>;
class Solution {
public:
    long long countMajoritySubarrays(vector<int>& nums, int target)
        int idx = 0; // 插入时自增，用来保证 st 中的元素互不相同
        ordered_set* st = new ordered_set();
        st->insert({0, ++idx});
        long long ans = 0;
        int s = 0;
        for (int x : nums) {
            s += x == target ? 1 : -1;
            // order_of_key(key) 计算 st 中的严格小于 key 的元素个数
            ans += st->order_of_key({s, 0});
            st->insert({s, ++idx});
        }
        delete st;
        return ans;
    }
};
```

**解释**：
#### 1. 先回顾你前面的需求
你要：**统计集合里严格小于某个值 X 的元素总个数**
- multiset /multimap 怎么做：
    1. `lower_bound(X)` 二分找到分界点，\(O(\log n)\)
    2. `distance(begin, it)` 从开头一步步迭代走到分界点，数有多少个元素，\(O(k)\)，k 是小于 X 的元素数量
        总复杂度：\(O(\log n + k)\)，最坏 \(O(n)\)

#### 2. ordered_set 底层独有的设计：节点存子树大小 sz
pb_ds 的树开启 `tree_order_statistics_node_update` 后，**每个节点会多存一个 sz 值**：
sz = 以该节点为根的子树一共有多少个节点。
插入、删除、旋转平衡时，会自动向上更新整条路径所有节点的 sz，全程只走树高 \(log n) 个节点，更新代价 \(O(log n)\)。


#### 3. 关键： order_of_key (X) 的执行流程（只走根到目标的一条路径，无遍历）
目标：算出有多少元素严格小于 X
从根节点向下二分遍历（路径长度 = 树高 \(log n)）：
1. 当前节点值 < X：
    所有左子树元素 + 当前节点本身 全都小于 X，累加 `cur->left.sz + 1`，去右子树继续找；
2. 当前节点值 ≥ X；
    当前节点和右子树都不算，直接往左子树走，不加任何计数；
3. 走到空节点停止，返回累加总和。
全程**只访问 (log n) 个节点**，不需要遍历所有小于 X 的元素，没有 (O(k) 那一步，复杂度稳定 (Olog n)。
