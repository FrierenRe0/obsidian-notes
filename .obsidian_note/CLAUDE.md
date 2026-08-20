# Obsidian Vault 项目记忆

## GitHub Markdown 渲染规则（KaTeX）

### 1. 被 GitHub 封禁的命令
- ❌ `\operatorname{cross}` → 显示源码
- ✅ 改用 `\text{cross}` 或 `\mathrm{cross}`

### 2. `$$...$$` 必须单行紧贴
- ✅ `$$\vec{a} \times \vec{b} = ...$$`（单行，`$$` 和内容之间无空格）
- ❌ `$$` 独占一行的多行写法 → 显示源码
- ❌ `$$ 内容 $$`（`$$` 后有空格）→ 显示源码
- ❌ `$$...$$` 后同一行还跟其他文字 → 显示源码

### 3. 全角标点不能紧贴 `$`
- ❌ `：$\vec{u}$`、`，$\vec{v}$`、`、$\vec{w}$` → 显示源码
- ✅ `： $\vec{u}$`、`， $\vec{v}$`、`、 $\vec{w}$`（加空格）
- 包括：`：`（冒号）、`，`（逗号）、`、`（顿号）、`。`（句号）等所有全角标点

### 4. `&` 字符在公式里完全不可用！
- ❌ `a\&b`、`a\mathrm{\&}b`、`a\mathbin{\&}b` → 全部 "Unable to render expression"
- ✅ 按位与改用 `\wedge`（∧）：`a \wedge b`
- ⚠️ 注意：代码块和 URL 里的 `&` 没问题，只有 LaTeX 公式里的 `&` 会崩

### 5. LaTeX 命令必须与后面字母隔开
- ❌ `a\oplusb` → 被解析成不存在的命令 `\oplusb` → 报错
- ✅ `a \oplus b`（`\oplus` 前后加空格）
- ❌ `a\wedgeb` → 同理
- ✅ `a \wedge b`

### 6. 其他坑
- ❌ `\left\vert{}`（空花括号）→ KaTeX 报错
- ✅ `\left\vert`（去掉 `{}`）
- ❌ `\begin{vmatrix}` 兼容性差
- ✅ `\left|\begin{matrix}...\end{matrix}\right|`
- ❌ `\vec{AB}`（箭头只盖一个字母）
- ✅ `\overrightarrow{AB}`
- ❌ `\hat{}`（空帽子）当异或符号 → 渲染怪异
- ✅ 异或用 `\oplus`（⊕）
- ❌ `\text{^}`（^ 是保留字符）→ 可能报错
- ✅ `\text{^}` 换成 `\oplus` 或 `\wedge`

### 7. 已验证可用的命令
`\vec`、`\overrightarrow`、`\text`、`\left\vert`、`\begin{aligned}`（单行）、`\left|\begin{matrix}`、`$...$`、单行`$$...$$`、`\oplus`（加空格）、`\wedge`（加空格）、`\mid`、`\gcd`、`\sim`

## Obsidian 图片路径

- 附件统一放 `截图文件/`（仓库根目录）
- 笔记中引用需写相对路径（`../../截图文件/...`）
- 已在 `app.json` 配置 `attachmentFolderPath: "截图文件"`
- 已配置 `newLinkFormat: "relative"`

## 需要 Git 忽略的内容

- `算法比赛常见单词/`（已加入根目录 `.gitignore`）
- `Obsidian Vault note/` 需要推送到 GitHub（不要忽略）
- 根目录 `.gitignore` 是唯一有效的忽略文件（git 根在 `个人仓库/`，不在 `.obsidian_note/`）

## 题卷 PDF

- 存放在 `题解/牛客/暑假训练营/2026/题卷/`
- 笔记中引用：`[第X场题卷](../题卷/第X场.pdf)`
- **上传/引用规则**：PDF 题卷必须与对应场次的比赛笔记放在同一大目录下（`题卷/` 与 `第X场/` 平级），笔记里的「题卷查看」链接要指向本地仓库内的 PDF 相对路径，**不要用浏览器/在线 URL 或本地 `file:///` 路径**
- 链接形式统一为 `[第X场题卷](../题卷/第X场.pdf)`，保证 GitHub 上可直接打开
- **主动连接规则**：一旦发现 `题卷/` 里有新 PDF，且对应场次笔记的「题卷查看」链接还没指向它（无论是 `file:///`、在线 URL 还是占位），**主动**把链接改为仓库内相对路径并同步到对应场次所有笔记，**不需要用户提醒**

## Markdown 换行

- 相邻两行没有空行 → 会被合并成同一段落（并排显示）
- 需要换行用：列表 `- ` 或空行分隔

## 用户指令标签

- **`pop(test)`**：用户打出这个标签时，需要删除 GitHub 上的所有测试样例文件（如 `test_*.md`、`测试数学.md`），从仓库中清理掉
- **测试文件提醒**：每次完成更改后，如果 GitHub 上还有测试样例文件未删除，都要主动提醒用户清理
