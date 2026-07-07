# AGENTS.md — Agent设计之美

## 项目定位

这是一个**学习型工作空间**，用于阅读技术教程、做笔记、写代码实验。

**同时也是一本 VitePress 博客**，部署后可在网上阅读课程笔记。

---

## 目录约定

```
/notes/              → 学习笔记草稿（各课程分目录）
  /notes/agent-design/   → Agent 设计之美 课程笔记
  /notes/next-course/    → 未来新课程
/demos/              → 代码实验，每个实验一个子目录
/raw/                → 课程原文 HTML（各课程分目录）
  /raw/agent-design/     → Agent 设计之美 原始 HTML
  /raw/next-course/     → 未来新课程
/references/          → 保存的外部参考材料（PDF、截图、网页存档等）
/docs/               → VitePress 博客源码（最终展示内容）
  /docs/.vitepress/config.js   → 主题配置（侧边栏、导航、搜索）
  /docs/index.md                → 博客首页
  /docs/public/                → 静态资源（封面图、favicon）
  /docs/courses/agent-design/  → Agent 设计之美 课程
    /docs/courses/agent-design/index.md    → 课程介绍页
    /docs/courses/agent-design/00-*.md   → 各讲笔记
    /docs/courses/agent-design/images/    → 课程图片
```

**新增课程时**，同时在 `notes/`、`raw/`、`docs/courses/` 下创建同名子目录，保持三个位置对齐。

---

## 博客使用说明

### 安装依赖

```bash
npm install
```

### 本地预览

```bash
npm run dev      # 开发服务器 http://localhost:5173
npm run build    # 构建静态文件到 docs/.vitepress/dist
npm run preview  # 预览构建结果
```

### 新增课程

在 `docs/courses/` 下创建新课程目录，参照 `agent-design/` 结构组织即可。然后在 `docs/.vitepress/config.js` 的 `nav` 和 `sidebar` 中追加新课程的入口。

### 新增讲次

在对应课程的目录下新建 `XX-标题.md` 文件，然后在 `config.js` 的 `sidebar` 中追加链接。

---

## 通用规范

参照全局 `CLAUDE.md`（位于 `/Users/wudandan/CLAUDE.md`）：
- 默认中文沟通，代码标识符用英文
- 结论先行，不谄媚
- 不确定时先确认再动手

---

## 工具链

博客：VitePress
笔记：Markdown + VitePress 的图片嵌入规范（相对路径 `./images/`）

## 学习工作流技能

处理新课程文章时，加载技能：

```
技能：course-learning-workflow
```

技能位置：`.skills/course-learning-workflow/SKILL.md`

覆盖：raw HTML → 结构化笔记 → VitePress 博客的完整流程。
