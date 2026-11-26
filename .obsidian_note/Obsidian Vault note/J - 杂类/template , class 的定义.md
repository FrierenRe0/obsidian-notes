# class

在 C++ 算法代码中，`class` 出现在不同位置时含义完全不同：

1. **尖括号里的 `class`（类型占位符）**：写在 `template<class T>` 中。这里的 `class` 与 `typename` **100% 等价**，意思是“任意数据类型”（可以是 `int`、`double`、`vector<int>` 等，不局限于类）。
    
2. **定义数据结构时的 `class`（类定义）**：写在 `class Node { ... };` 中。它和 `struct` 属于同一类语法，用来把数据（成员变量）和操作（成员函数）打包在一起。

## **注意**：class使用场景


在 C++ 中，`struct` 和 `class` 语法结构完全一致，唯一的区别在于**默认访问权限**：`struct` 默认全部公开（`public`），而 `class` 默认全部私有（`private`）。
- 补充 `private` : 对于`private`定义的成员，**当前结构体自己的所有成员（无论写在 `public:` 还是 `private:` ）都可以直接访问`private`定义的数据**；而**外部的任何代码（包括其他 `struct` 的 `public` 区域）默认都无法直接访问**

下面通过“打包数据节点”**和**“封装数据结构”两个典型算法场景进行直接对比。

### 场景一：打包数据节点（如图的边、区间、坐标点）

在这种场景下，主函数需要频繁直接读写内部变量，并用 `sort` 排序。

#### 1. `struct` 写法（默认 `public`，开箱即用）

C++

```
struct Edge {
    int u, v, w; // 默认 public，外部可直接读写

    // 重载小于号用于 sort
    bool operator<(const Edge &other) const {
        return w < other.w;
    }
};

// 主函数调用：
Edge e = {1, 2, 10}; // 支持聚合初始化
e.w = 20;            // ✅ 合法：直接修改成员变量
cout << e.u << '\n'; // ✅ 合法：直接读取成员变量
```

#### 2. `class` 写法（必须手动加 `public:`）

C++

```
class Edge {
public: // ⚠️ 必须显式写出 public:，否则下面所有成员和构造函数全部失效
    int u, v, w;

    Edge(int u, int v, int w) : u(u), v(v), w(w) {}

    bool operator<(const Edge &other) const {
        return w < other.w;
    }
};

// 主函数调用：
Edge e(1, 2, 10);
e.w = 20;            // ✅ 加了 public: 之后才能访问
```

> **注意**：如果在 `class Edge` 里漏写了 `public:`，不仅 `e.w` 无法访问，连构造函数都会变成私有的，导致外部根本无法创建 `Edge` 对象（直接编译报错）。

### 场景二：封装数据结构（如并查集 `DSU`）

在这种场景下，数据结构内部维护着核心状态数组（如父节点数组 `fa`），外部只需要调用接口函数（`find`、`merge`）。

#### 1. `struct` 写法（全开放，方便调试）

C++

```
struct DSU {
    vector<int> fa, sz;

    DSU(int n) : fa(n + 1), sz(n + 1, 1) {
        iota(fa.begin(), fa.end(), 0);
    }

    int find(int x) {
        return fa[x] == x ? x : fa[x] = find(fa[x]);
    }

    bool merge(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return false;
        fa[x] = y;
        sz[y] += sz[x];
        return true;
    }
};

// 主函数调用：
DSU dsu(10);
dsu.merge(1, 2);
cout << dsu.sz[dsu.find(1)] << '\n'; // ✅ 调试时可直接查看内部数组
```

#### 2. `class` 写法（隐藏内部状态，仅暴露安全接口）

C++

```
class DSU {
private: // 默认就是 private，这里写出来是为了结构清晰
    vector<int> fa, sz; // 核心数组被保护，外部禁止直接篡改

public:  // 从这一行开始，直到遇到下一个权限关键字前，全部对外公开
    DSU(int n) : fa(n + 1), sz(n + 1, 1) {
        iota(fa.begin(), fa.end(), 0);
    }

    int find(int x) {
        return fa[x] == x ? x : fa[x] = find(fa[x]);
    }

    bool merge(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return false;
        fa[x] = y;
        sz[y] += sz[x];
        return true;
    }

    // 既然 sz 是私有的，就必须提供一个专门的查询接口给外部用
    int get_size(int x) {
        return sz[find(x)];
    }
};

// 主函数调用：
DSU dsu(10);
dsu.merge(1, 2);
// dsu.fa[1] = 5;              // ❌ 编译报错：防止手误破坏并查集结构
cout << dsu.get_size(1) << '\n'; // ✅ 只能通过 public 接口安全获取大小
```

### 核心差异速记

1. **权限关键字是“开关”**：无论是 `struct` 还是 `class`，内部都可以混用 `public:` 和 `private:`。关键字一旦出现，就会一直生效，直到遇到下一个权限关键字或大括号 `};` 结束。
    
2. **外部排序函数的权限限制**：如果用 `class` 定义节点且把权重 `w` 放在 `private:` 里，写在类外面的 `bool cmp(const Node &a, const Node &b)` 会因为无权读取 `a.w` 而报错；此时只能把 `operator<` 写在类的 `public:` 内部。
    
3. **实战选型**：刷题写模板时直接用 `struct`，省去写 `public:` 和 `get_xxx()` 辅助函数的代码量；只有在封装大型工程模块或希望严格防止内部状态被误改（如自动取模类 `Mint`）时，才使用 `class` 隔离内部变量


### `private`定义函数的优势 ： 

把函数放进 `private:` 后，它就变成了**内部辅助函数（Helper Function）**——**只有当前结构体自己的其他成员函数（以及 `friend`）能调用它**，在主函数 `main()` 或其他结构体外部直接调用会直接编译报错。

在算法数据结构（如线段树、平衡树）中，把函数放进 `private:` 非常实用，主要有两大目的：

1. **隐藏底层维护逻辑，防止手误破坏结构**：例如线段树的 `pushup`（上传信息）和 `pushdown`（下传懒标记），它们只应该在递归过程中自动触发，外部在 `main` 里手动调用没有任何意义，反而容易破坏数据一致性。
    
2. **给多参数递归函数套一层“简洁外壳”**：递归核心通常需要传入当前节点编号和区间端点 `(p, l, r)`，把复杂的递归版本放进 `private:`，再在 `public:` 里提供一个只需要传目标区间 `(ql, qr)` 的同名重载函数，主函数调用时既简洁又不会填错根节点参数。
    

### 线段树样例：内外分离写法

C++

```
struct SegTree {
private:
    int n;
    vector<int> sum, tag;

    // 1. 底层维护函数：放 private，外部不可见
    void pushup(int p) {
        sum[p] = sum[p << 1] + sum[p << 1 | 1];
    }

    void pushdown(int p, int l, int r) {
        if (!tag[p]) return;
        int mid = (l + r) >> 1;
        sum[p << 1] += tag[p] * (mid - l + 1);
        sum[p << 1 | 1] += tag[p] * (r - mid);
        tag[p << 1] += tag[p];
        tag[p << 1 | 1] += tag[p];
        tag[p] = 0;
    }

    // 2. 内部递归核心：参数多，放 private 避免外部调用时传错 p, l, r
    void add(int p, int l, int r, int ql, int qr, int v) {
        if (ql <= l && r <= qr) {
            sum[p] += v * (r - l + 1);
            tag[p] += v;
            return;
        }
        pushdown(p, l, r); // ✅ 合法：private 函数之间可以互相调用
        int mid = (l + r) >> 1;
        if (ql <= mid) add(p << 1, l, mid, ql, qr, v);
        if (qr > mid) add(p << 1 | 1, mid + 1, r, ql, qr, v);
        pushup(p);
    }

public:
    SegTree(int n) : n(n), sum(n * 4 + 1), tag(n * 4 + 1) {}

    // 3. 对外暴露的简洁接口：内部转发给 private 递归函数
    void add(int ql, int qr, int v) {
        add(1, 1, n, ql, qr, v); // ✅ 合法：public 函数可以调用 private 函数
    }
};
```



## template

### 1. 类模板（构建通用数据结构板子）

把 `template<class T>` 写在 `struct` 上方，整个结构体内部都可以把 `T` 当作一种具体类型来使用。声明对象时需要在尖括号里手动指定类型。

C++

```
template<class T>
struct BIT {
    int n;
    vector<T> w;
    BIT(int n) : n(n), w(n + 1, T{}) {} // T{} 表示 T 类型的零值

    void add(int x, T v) {
        for (; x <= n; x += x & -x) w[x] += v;
    }
    T ask(int x) {
        T res{};
        for (; x > 0; x -= x & -x) res += w[x];
        return res;
    }
};

// 调用方式：
BIT<int> bit1(100);        // 维护 int
BIT<long long> bit2(100);  // 维护 long long
```

### 2. 函数模板（通用工具函数）

把 `template<class T>` 写在函数上方。调用时不需要写尖括号，编译器会根据传入的实参**自动推导** `T` 的类型。

C++

```
template<class T>
bool chmin(T &a, const T &b) {
    return b < a ? (a = b, true) : false;
}

template<class T>
bool chmax(T &a, const T &b) {
    return a < b ? (a = b, true) : false;
}

// 调用方式：直接传参，编译器自动推导 T 为 int 或 long long
int x = 10;
chmin(x, 5); 
```

> **C++20 简写技巧**：在开启 C++20 及以上标准的 OJ 中，函数模板可以直接用 `auto` 代替，例如 `bool chmin(auto &a, const auto &b)`，甚至允许 `a` 为 `long long` 而 `b` 为 `int`（混合类型比较）。

### 3. 成员函数模板（局部类型适配）

结构体本身不是模板，但内部的某个函数（常见于构造函数）需要接收多种容器或兼顾 `const` 与非 `const` 参数。作用域**仅限当前这一个函数**。

C++

```
struct BIT {
    int n;
    vector<int> w, chk;

    // 局部模板：V 只在这个构造函数里有效
    template<class V> 
    BIT(const V &in) : n(in.size() - 1), w(in.size()), chk(in.begin(), in.end()) {}
    
};
```

- 加上 `const` 后，既能传入普通变量 `BIT b(a)`，也能传入临时右值 `BIT b(vector<int>{0, 3, 1, 2})`。
    
- 使用迭代器区间 `chk(in.begin(), in.end())` 初始化，可以真正兼容 `vector`、`array`、`deque` 等不同容器。