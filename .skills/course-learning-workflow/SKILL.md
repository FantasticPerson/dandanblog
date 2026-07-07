---
name: course-learning-workflow
description: Use when processing a new course article from GeekBang (极客时间) or similar platform into structured Markdown notes with embedded images, or when setting up a VitePress blog for a learning workspace. Triggers: user pastes raw HTML, asks to "整理" a lecture, or says "创建博客" for a course.
---

# 课程学习笔记工作流

将原始课程文章整理成结构化 Markdown 笔记，并嵌入图片，最终输出 VitePress 博客格式。

## 核心原则

**先建秩序，再填内容。** 每篇笔记必须包含：
1. Frontmatter（title + 一句话脉络）
2. 按文章结构的 Markdown 标题层级
3. 每张图片的相对路径嵌入

## 标准化目录结构

每门课程在三个位置保持对齐：

```
raw/{course}/           ← 原始 HTML（备用）
notes/{course}/         ← 整理好的 Markdown 笔记 + images/
docs/courses/{course}/ ← VitePress 博客 + images/（定期从 notes 复制）
```

新增课程时，**同时**在 `raw/`、`notes/`、`docs/courses/` 下创建同名子目录。

## 标准流程（按顺序执行）

### Step 1：提取文本

极客时间使用 Slate 编辑器，文本在 `data-slate-string="true"` 的 span 里。

```python
import re
with open('raw_file', 'r') as f:
    content = f.read()
texts = re.findall(r'data-slate-string="true"[^>]*>([^<]+)</span>', content)
full_text = '\n'.join(texts)
```

### Step 2：提取图片 URL

```python
urls = re.findall(r'https://static001\.geekbang\.org/[^\s"\'<>]+\.(?:jpg|png|webp|svg)(?:\?wh=[^"\']*)?', content)
for u in urls:
    print(u)
```

### Step 3：下载图片

```bash
cd "notes/{course}/images"
curl -sL -o "NN-01.jpg" "https://static001.geekbang.org/resource/image/xx/xx/xxx.jpg?wh=..."
```

命名规则：`{讲次}-{序号}.{ext}`，如 `03-01.jpg`、`12-03.png`。

### Step 4：创建 Markdown 笔记

每篇笔记的 YAML frontmatter：

```yaml
---
title: NN｜文章标题
---
```

正文结构：

```
# NN｜文章标题

**作者**：黄佳（或其他）

---

## 一句话脉络
本讲核心问题的一句话总结

---

## [主体结构按文章内容组织]
```

嵌入图片：`![描述](./images/NN-01.jpg)`

### Step 5：同步到 VitePress

```bash
# 复制笔记到 docs
cp "notes/{course}/"*.md "docs/courses/{course}/"
# 复制图片
cp -r "notes/{course}/images/" "docs/courses/{course}/"
```

图片路径 `./images/...` 在两个目录下都有效。

### Step 6（可选）：初始化 VitePress 博客

仅在课程**首次**建博客时执行：

1. 创建 `docs/.vitepress/config.js`（导航 + 侧边栏）
2. 创建 `docs/index.md`（首页 Hero）
3. 创建 `docs/courses/{course}/index.md`（课程介绍页）
4. 创建 `package.json`（含 `"type": "module"` + VitePress scripts）
5. `npm install && npm run dev`

侧边栏按讲次排序，分模块分组。

## 图片嵌入原则

- 在语义边界处插入（如章节标题后）
- 用描述性 alt 文字，不用"图片"
- 不确定位置时，放在相关章节的第一个 H2 之后

## 判断是否需要建博客

| 条件 | 行动 |
|---|---|
| 课程有多讲、需要长期积累 | 建 VitePress 博客 |
| 单篇文章、快速笔记 | 只建 `notes/` 即可 |

## 常见错误

| 错误 | 原因 | 修复 |
|---|---|---|
| 图片显示不出来 | 目标目录没有 images/ | 先 `mkdir -p docs/courses/{course}/images/` 再复制 |
| 笔记路径找不到 | 中文文件名编码问题 | 用 `-o "NN-01.jpg"` 指定输出名 |
| 文本提取乱码 | HTML entity 未解码 | 用 HTML parser 而非正则直接剥标签 |
| 讲次顺序错 | 文件名没用两位序号 | `01`、`02` 而非 `1`、`2` |

## GeekBang 特定注意事项

- 极客时间用 Vue SPA，curl 只能拿 HTML骨架，需 Playwright 或用户粘贴原文
- 付费文章：用户粘贴内容，或提供 cookies/headers 用 curl 拿
- 免费文章：Playwright MCP 可直接抓取
- 图片 URL 需从 HTML 中用正则匹配，`?wh=xxx` 是缩放参数，可省略
