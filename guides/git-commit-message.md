---
title: 'Git 提交信息(Commit Message)怎么写'
pubDate: 2026-08-03
description: '解释 Git commit message 的格式规范、Conventional Commits 约定、常见 type、写法示例'
author: '学习笔记'
tags: ["git", "commit", "规范"]
---

# Git 提交信息(Commit Message)怎么写

## 一句话作用

commit message 是你给这次代码改动起的"标题+说明",方便以后回看历史、查找某次改了什么、为什么改。

## 生活比喻

把代码仓库想象成一本日记本,每次提交就是写一篇日记:

- **commit message** = 日记的标题 + 正文
- 写得好 → 以后翻日记能立刻知道某天干了什么
- 写得差(比如"update"、"fix bug")→ 以后翻日记一脸懵,"我那天到底改了啥?"

## 业界主流写法:Conventional Commits(约定式提交)

### 是什么

Conventional Commits 是一套**业界通用的 commit message 格式规范**,很多开源项目和公司团队都用它。

### 基本格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 各部分解释

| 部分 | 必填吗 | 作用 | 例子 |
|---|---|---|---|
| `type` | 必填 | 改动的类型(加了功能?修了 bug?) | `feat`、`fix`、`chore` |
| `scope` | 可选 | 改动影响的模块/范围 | `(posts)`、`(config)` |
| `subject` | 必填 | 一句话简述改了什么 | `新增博客文章页面` |
| `body` | 可选 | 详细说明为什么改、怎么改 | 多行文字 |
| `footer` | 可选 | 关联 issue、标记 BREAKING CHANGE | `Closes #123` |

### 例子

最简形式(只有 type + subject):
```
feat: 新增小红书学习笔记文章
```

带 scope:
```
feat(posts): 新增小红书学习笔记文章
```

带 body(详细说明):
```
feat(posts): 新增小红书学习笔记文章

- 创建 xiaohongshu-data-note.mdx 文件
- 添加 BlogPost.astro 布局组件
- 配置 mdx 集成支持
```

## 常见的 type(类型)

这是最重要的部分,记住这几个常用的就够:

| type | 含义 | 什么时候用 | 例子 |
|---|---|---|---|
| `feat` | feature(新功能) | 新增了功能、新页面、新文件 | 新增一篇博客文章 |
| `fix` | bug 修复 | 修了某个错误 | 修复日期显示成 [object Object] |
| `docs` | 文档改动 | 改了 README、注释、文档 | 更新 README |
| `style` | 代码格式 | 改缩进、空格、分号(不影响逻辑) | 格式化代码 |
| `refactor` | 重构 | 改了代码结构但不影响功能 | 把函数拆分 |
| `chore` | 杂务 | 构建、依赖、配置等非代码改动 | 升级依赖、改 config |
| `test` | 测试 | 加/改测试代码 | 加单元测试 |
| `perf` | 性能优化 | 提升性能 | 优化图片加载 |
| `ci` | CI 配置 | 改 GitHub Actions 等 | 配置自动部署 |
| `revert` | 撤销 | 撤销之前的提交 | 回滚某次改动 |

## 写 subject(标题)的几条规则

1. **用祈使句**(像下命令一样):
   - ✅ `新增博客文章`(像说"给我新增")
   - ❌ `新增了博客文章`(不要用过去时)

2. **首字母不大写**(英文的话)、结尾不加句号:
   - ✅ `add blog post`
   - ❌ `Add blog post.`

3. **简短**:50 字以内最好,超过的话考虑放 body 里

4. **说"改了什么",不说"怎么改"**:
   - ✅ `新增小红书笔记文章`(说了改什么)
   - ❌ `用 mdx 写了一篇文章并配上 layout`(说了怎么改,太啰嗦)

## 写 body(正文)的几条规则

1. 解释**为什么**改,不是**什么**改了(标题已经说了什么)
2. 每行不超过 72 字符(终端显示美观)
3. 用 `-` 列点更清晰

例子:
```
feat(posts): 新增小红书学习笔记文章

文章内容是关于 Python 装饰器学习的疑问点。
使用 mdx 格式,支持在 markdown 里插入组件。
layout 用 BlogPost.astro 统一渲染标题、日期、导航。

Closes #5
```

## footer(脚注)的用途

### 1. 关联 issue

```
Closes #123      ← 自动关闭 issue #123
Fixes #45        ← 修复了 issue #45
Resolves #67     ← 解决了 issue #67
```

GitHub 看到这些关键字会**自动关闭对应的 issue**。

### 2. 标记破坏性改动

```
BREAKING CHANGE: 文章 layout 从 .jsx 改为 .astro,旧 mdx 需更新 layout 路径
```

`BREAKING CHANGE` 是个特殊关键字,告诉别人这次改动**会让旧代码不兼容**,需要手动调整。

## 多个改动怎么提交?

有两种策略:

### 策略 1:一次提交全部(简单项目可以)

```bash
git add .
git commit -m "feat: 新增小红书笔记文章及 layout"
```

### 策略 2:分多次提交(推荐,更清晰)

把不同类型的改动分开提交:

```bash
# 第一次:只提交配置改动
git add astro.config.mjs package.json pnpm-lock.yaml
git commit -m "chore: 添加 @astrojs/mdx 集成"

# 第二次:提交 layout 组件
git add src/layouts/
git commit -m "feat(layouts): 新增 BlogPost.astro 文章布局"

# 第三次:提交文章内容
git add src/pages/posts/xiaohongshu-data-note.mdx
git commit -m "feat(posts): 新增小红书学习笔记文章"

# 第四次:提交文档
git add guides/
git commit -m "docs: 新增 layout 和 commit message 学习笔记"
```

分多次的好处:以后翻历史能清楚看到"先加了集成,再加 layout,再加文章",每一步都有明确目的。

## HEREDOC 语法(写多行 commit message)

如果 commit message 有 body,需要多行,用 HEREDOC 语法:

```bash
git commit -m "$(cat <<'EOF'
feat(posts): 新增小红书学习笔记文章

- 创建 xiaohongshu-data-note.mdx
- 配置 mdx 集成
- 添加 BlogPost.astro 布局
EOF
)"
```

`<<'EOF'` 和 `EOF` 之间的内容会作为完整字符串传给 `-m`。这种写法在 PowerShell、Bash、Zsh 都能用。

## 查看提交历史

写完提交后,用这些命令检查:

```bash
git log --oneline        # 简略查看(只看标题)
git log -5              # 查看最近 5 次提交
git show HEAD           # 查看最新一次提交的详细内容
git diff HEAD~1         # 跟上一次提交对比差异
```

## 推送到 GitHub

提交完只是在本地,还要推送到 GitHub:

```bash
git push                # 推到默认远程分支
git push origin main    # 明确指定推到 origin 的 main 分支
```

如果是第一次推送一个新分支:
```bash
git push -u origin 分支名    # -u 设置上游跟踪
```

## 替代方案:不用 Conventional Commits

不是所有人都用这套规范。也有别的写法:

### 1. 自由格式

```
新增小红书笔记文章
```

简单直接,但不规范,团队协作时不利于自动化工具。

### 2. GitHub 风格(用 #issue 号)

```
Add xiaohongshu note post (#5)
```

适合每个改动都对应一个 issue 的项目。

### 3. Angular 规范

比 Conventional Commits 更严格,要求 type、scope、subject 全部小写,subject 不超 72 字符等。

## 总结

- 用 Conventional Commits 格式:`type(scope): subject`
- 常用 type:`feat`、`fix`、`chore`、`docs`
- subject 用祈使句、简短、不加句号
- body 解释"为什么"改,不是"改了什么"
- 多个改动分多次提交更清晰
- 多行 message 用 HEREDOC 语法

## 你这次的具体情况

你项目当前的改动:

1. 删除了 README.md
2. 改了 astro.config.mjs(加了 mdx 集成)
3. 改了 package.json(加了 @astrojs/mdx 依赖)
4. 改了 pnpm-lock.yaml(锁文件更新)
5. 改了 src/pages/blog.astro(加了文章列表项)
6. 新增 guides/ 文件夹(学习笔记文档)
7. 新增 src/layouts/BlogPost.astro(layout 组件)
8. 新增 src/pages/posts/props.mdx(学习笔记)
9. 新增 src/pages/posts/xiaohongshu-data-note.mdx(博客文章)

建议分 3-4 次提交:

```bash
# 1. 配置改动
git add astro.config.mjs package.json pnpm-lock.yaml
git commit -m "chore: 添加 @astrojs/mdx 集成支持"

# 2. 删除 README 和改 blog.astro
git add README.md src/pages/blog.astro
git commit -m "chore: 删除 README,博客列表新增文章链接"

# 3. layout 和文章
git add src/layouts/ src/pages/posts/xiaohongshu-data-note.mdx
git commit -m "feat: 新增小红书学习笔记文章及 BlogPost 布局"

# 4. 学习文档
git add guides/
git commit -m "docs: 新增 layout、commit message 学习笔记"

# 5. 多余的 props.mdx(可选,如果不需要可以删掉)
```

如果觉得太麻烦,也可以一次性提交:
```bash
git add .
git commit -m "feat: 新增小红书学习笔记文章及 mdx 集成配置"
```
