**[703. 数据流中的第 K 大元素](https://leetcode.cn/problems/kth-largest-element-in-a-stream/) --> 一般堆处理 （底层为二叉树数据结构即可）**

这里我刚开始的时候是用了multiset数据结构的思考方向：
因为我觉得这里想找到一个top-k大的数字，我可以用一个数据类型将所用的数据排列成单调形式，再找到top-k大的数据即可；

### 代码如下：(同时标注了我方法的主要问题)
```c++
class KthLargest {
public:
    multiset<int> s;
    int w = -1;
    KthLargest(int k, vector<int>& nums) {
        for(int i = 0;i < nums.size();i++) {
            s.insert(nums[i]);
        }
        w = k;
    }

    int add(int val) {
        s.insert(val);
        //这里做添加的时间复杂度为logn;同时这里也会有个隐患就是当我再不断反复调用add函数从而增加了数据量，则每次再增加数据时是一次比一次慢的；（次）
        auto it = s.begin();
        advance(it,s.size() - w);
        //这里才是我代码慢的主要原因，因为我这里去找到top-k大数据时本质就是个暴力查找，但是时间复杂度暴增
        return *it;
    }
};

/**
 * Your KthLargest object will be instantiated and called as such:
 * KthLargest* obj = new KthLargest(k, nums);
 * int param_1 = obj->add(val);
 */
```

**现在的优化方向就是我因该如果去做到 logn 又或者是 0(1)的查询，同时每次添加数据时的时间复杂度稳定 ？**

**思考**：
理解题目会发现一个特点，就是当我想要找到一个数据结构维护从而完成找到top-k大的数据时，这里一定时一个包含内排序的类型，同时，同时在数据流中维护前k大数据，这里可以用priority_queue。

**步骤**：
1.当队列中的数据量还没到k时，直接push；
2.否则，如果当前我所添加的数据，还没有加入队列的实力（也是时当前数字x <=  pq.top()）时，我可以跳过不处理；如果又实力（也是时当前数字x  >  pq.top()）时。队列里面最小的数据也就是pq.top() --> 因为我这里时小根堆要pop(),在把我当前元素push,从而在数据流中完美的维护了当前的前k个元素
3.这里就完成了 0(1)查询，logn添加数据；
4.其实这里也可以用multiset只是不可以和上面那样写，因为其底层逻辑和堆本质时类似的

### priority_queue实现代码如下：
```c++
class KthLargest {
public:
    priority_queue<int,vector<int>,greater<>> pq;
    int K;
    KthLargest(int k, vector<int>& nums) {
        K = k;
        for(int i = 0;i < nums.size();i++) {
            if(pq.size() < k) pq.push(nums[i]);
            else {
                if(nums[i] > pq.top()) {
                    pq.pop();
                    pq.push(nums[i]);
                }
            }
        }
    }

    int add(int val) {
        if(pq.size() < K) pq.push(val);
        else {
            if(val > pq.top()) {
                pq.pop();
                pq.push(val);
            }
        }
        return pq.top();
    }
};
/**
 * Your KthLargest object will be instantiated and called as such:
 * KthLargest* obj = new KthLargest(k, nums);
 * int param_1 = obj->add(val);
 */
```

**multiset实现代码如下：(因为底层逻辑时一样的，所以和priority_queue的代码类似)**
```c++
class KthLargest {
public:
    multiset<int> s;
    int w;
    KthLargest(int k, vector<int>& nums) {
        for(int i = 0;i < nums.size();i++) {
            if(s.size() < k) s.insert(nums[i]);
            else {
                if(nums[i] > *s.begin()) {
                    s.erase(s.begin());
                    s.insert(nums[i]);
                }
            }
        }
        w = k;
    }

    int add(int val) {
        if(s.size() < w) s.insert(val);
        else {
            if(val > *s.begin()) {
                s.erase(s.begin());
                s.insert(val);
            }
        }
        return *s.begin();
    }
};

/**
 * Your KthLargest object will be instantiated and called as such:
 * KthLargest* obj = new KthLargest(k, nums);
 * int param_1 = obj->add(val);
 */
```
