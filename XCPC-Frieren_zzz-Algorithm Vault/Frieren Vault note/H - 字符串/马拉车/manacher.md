```c++
string longestPalindrome(string s) {
    if (s.empty()) return "";

    // 1. 预处理字符串：在字符间及首尾插入特殊字符（如 '#' 和边界符），统一奇偶回文处理
    string t = "^";
    for (char c : s) {
        t += "#";
        t += c;
    }
    t += "#$";

    int n = t.size();
    vector<int> p(n, 0); // p[i] 表示以 i 为中心的回文半径（包含 i 本身）
    int id = 0, mx = 0;  // id 为当前右边界最大的回文子串的中心，mx 为其右边界 (id + p[id])
    int max_len = 0;     // 记录原字符串中最长回文子串的长度
    int center_index = 0;// 记录最长回文子串在新字符串 t 中的中心位置

    // 2. 核心计算过程
    for (int i = 1; i < n - 1; i++) {
        // 利用对称性初始化 p[i]
        p[i] = (i < mx) ? min(mx - i, p[2 * id - i]) : 1;

        // 尝试中心扩展
        while (t[i - p[i]] == t[i + p[i]]) {
            p[i]++;
        }

        // 如果新回文串的右边界超过了 mx，更新 id 和 mx
        if (i + p[i] > mx) {
            id = i;
            mx = i + p[i];
        }

        // 记录全局最大回文长度及中心
        if (p[i] - 1 > max_len) {
            max_len = p[i] - 1;
            center_index = i;
        }
    }

    // 3. 根据 t 中的中心和长度还原回原字符串中的起始位置
    int start = (center_index - max_len) / 2;
    return s.substr(start, max_len);
}
```