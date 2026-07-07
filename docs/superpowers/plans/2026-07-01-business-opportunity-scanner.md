# 商业机会扫描器 — 实现计划

> 使用子代理驱动开发（subagent-driven-development）逐任务实现此计划。

**目标：** 构建一个自动采集多源信息、分析商业可行性的全栈 Web 应用
**架构：** NestJS + MongoDB 后端（含采集和分析），Vue3 + Vite 前端，Bull + Redis 异步队列
**技术栈：** NestJS, Mongoose, Bull, Redis, Playwright, Cheerio, DeepSeek API, Vue3, ECharts

---

### 任务 1：项目脚手架搭建

**文件：**
- 创建：`server/`（NestJS 项目）
- 创建：`server/package.json`
- 创建：`client/`（Vue3 + Vite 项目）
- 创建：`client/package.json`
- 创建：`docker-compose.yml`（MongoDB + Redis）

- [ ] **搭建 NestJS 后端项目**
  使用 `nest new` 或手动创建 NestJS 项目结构，配置 MongoDB 连接（`@nestjs/mongoose` + `mongoose`）、Bull（`@nestjs/bull`）

- [ ] **搭建 Vue3 + Vite 前端项目**
  使用 `create-vite` 创建 Vue3 + TS 项目，安装 ECharts 依赖

- [ ] **创建 docker-compose.yml**
  包含 `mongodb` 和 `redis` 两个 service

- [ ] **Commit**

### 任务 2：数据模型（Mongoose Schemas）

**文件：**
- 创建：`server/src/schemas/project.schema.ts`
- 创建：`server/src/schemas/raw-page.schema.ts`
- 创建：`server/src/schemas/analysis-report.schema.ts`
- 创建：`server/src/schemas/source-status.schema.ts`

- [ ] **定义 4 个 Schema**

根据设计文档的数据模型定义，Mongoose Schema + TypeScript 接口。

关键实现要点：
- `Project`：`source` 枚举 manual/auto，`status` 枚举 draft/collecting/analyzing/completed/failed
- `RawPage`：`source` 枚举 7 个信源，`metrics` 为混合类型对象
- `AnalysisReport`：`dimensions` 下 6 个子维度，每个包含 score(1-10)/analysis/evidence
- `SourceStatus`：用于采集源健康监控

- [ ] **Commit**

### 任务 3：后端 REST API

**文件：**
- 创建：`server/src/projects/projects.module.ts`
- 创建：`server/src/projects/projects.controller.ts`
- 创建：`server/src/projects/projects.service.ts`

- [ ] **实现 API 端点**

| 端点 | 方法 | 功能 |
|---|---|---|
| `/api/projects` | GET | 项目列表，支持 status/source 筛选、排序、分页 |
| `/api/projects` | POST | 新增项目（手动输入想法） |
| `/api/projects/:id` | GET | 项目详情（含原始数据和报告） |
| `/api/projects/:id` | DELETE | 删除项目 |
| `/api/projects/:id/analyze` | POST | 手动触发分析 |
| `/api/sources/status` | GET | 各采集源状态 |

- [ ] **Commit**

### 任务 4：采集调度系统

**文件：**
- 创建：`server/src/collector/collector.module.ts`
- 创建：`server/src/collector/collector.service.ts`
- 创建：`server/src/collector/collector.processor.ts`（Bull Consumer）
- 创建：`server/src/collector/sources/product-hunt.ts`
- 创建：`server/src/collector/sources/github-trending.ts`
- 创建：`server/src/collector/sources/thirty-six-kr.ts`
- 创建：`server/src/collector/sources/hacker-news.ts`
- 创建：`server/src/collector/sources/reddit.ts`
- 创建：`server/src/collector/sources/twitter.ts`
- 创建：`server/src/collector/sources/app-store.ts`

- [ ] **实现队列基础设施**
  Bull Queue 配置，队列名 `collection-queue`，支持并发控制

- [ ] **实现 7 个信源 Worker**

每个 Worker：
- 接收任务类型（`source_name` + 可选关键词）
- 爬取对应源的最新数据
- 去重（查 MongoDB 已有 url）
- 写入 raw_pages，创建或关联到已有 project
- 完成后触发分析队列

采集方式参考：
- Product Hunt：Playwright 打开当日产品页，提取标题/描述/votes
- GitHub Trending：Playwright 抓 trending 页
- 36氪：Playwright 抓科技快讯列表
- Hacker News：`https://hacker-news.firebaseio.com/v0/topstories.json`
- Reddit：`https://www.reddit.com/r/SideProject/.rss`
- Twitter/X：使用开源 RSS 代理（如 nitter）
- App Store：`https://rss.applemarketingtools.com/api/v2/cn/apps/top-free/50/apps.json`

- [ ] **实现 Scheduler**
  每天早 8 点 / 晚 8 点触发全源采集（node-cron 或 Bull Queue 的 repeatable job）

- [ ] **Commit**

### 任务 5：分析评估模块

**文件：**
- 创建：`server/src/analyzer/analyzer.module.ts`
- 创建：`server/src/analyzer/analyzer.service.ts`
- 创建：`server/src/analyzer/analyzer.processor.ts`（Bull Consumer）
- 创建：`server/src/analyzer/prompts.ts`

- [ ] **实现分析队列**

监听 `collection-queue` 完成事件，自动将新数据入队 `analysis-queue`

- [ ] **实现 LLM 分析服务**

分析 prompt 结构：

```
你是一个商业分析师。分析以下项目信息，从 6 个维度评分（1-10）。

项目信息：
{raw_pages 摘要 + 用户想法}

维度：
1. 市场分析：规模、增长、竞争格局
2. 技术实现难度：需要什么技术栈、开发周期
3. 商业化潜力：收费模式、目标用户
4. 合规风险：国内能不能做、需要什么资质
5. 出海可行性：海外市场机会
6. 个人开发者友好度：启动成本

每个维度输出：分数 + 分析理由 + 证据来源引用
综合输出：加权总分 + 一句话结论（推荐/观望/不推荐）+ 风险提醒列表
```

- [ ] **实现报告落地**
  解析 LLM 返回的 JSON，写入 `analysis_reports`，更新 `project.status = 'completed'`

- [ ] **Commit**

### 任务 6：前端仪表盘

**文件：**
- 创建：`client/src/views/Dashboard.vue`
- 创建：`client/src/components/StatCard.vue`
- 创建：`client/src/components/ProjectCard.vue`
- 创建：`client/src/components/RadarChart.vue`
- 创建：`client/src/components/TrendChart.vue`
- 创建：`client/src/components/WordCloud.vue`
- 创建：`client/src/components/SourcePieChart.vue`
- 创建：`client/src/api/index.ts`

- [ ] **实现 API 请求层**

使用 `fetch` 或 `axios` 封装所有后端 API 调用

- [ ] **实现仪表盘页面**

布局：
- 顶部：搜索框 + "新增想法"按钮 + "刷新采集"按钮
- 统计卡区：总项目 / 本周新增 / 待分析 / 平均分
- 推荐 Top 5：卡片列表，点击进入详情
- 趋势看板（ECharts）：
  - 雷达图：各维度评分分布（可选项目对比）
  - 柱状图：每周/月新增项目数
  - 饼图：各信源采集占比
  - 词云：热门标签

- [ ] **Commit**

### 任务 7：前端项目详情页

**文件：**
- 创建：`client/src/views/ProjectDetail.vue`

- [ ] **实现详情页**

布局：
- 项目标题 + 来源标签 + 采集时间
- 综合评分大数字 + 结论标签（推荐/观望/不推荐）
- 风险提醒（红色警告条）
- 六维评分柱状图（每个维度横向柱 + 分数数字）
- 可展开的分析详情（点击维度展开 LLM 分析正文 + 引用来源）
- 原始采集数据列表（标题/来源/链接/摘要）

- [ ] **Commit**

### 任务 8：前端项目管理页

**文件：**
- 创建：`client/src/views/ProjectList.vue`

- [ ] **实现列表页**

布局：
- "新增想法"按钮 + "批量分析"
- 筛选标签：全部/待采集/待分析/已完成/失败
- 排序：综合评分/来源/时间
- 项目列表行：标题、评分、状态标签、时间、操作（删除/重分析）

- [ ] **Commit**

### 任务 9：新增想法对话框 + 手动触发采集

**文件：**
- 创建：`client/src/components/NewProjectDialog.vue`
- 修改：`server/src/projects/projects.service.ts`

- [ ] **实现新增想法的对话框/页面**

表单字段：
- 项目标题（必填）
- 项目描述（必填）
- 可选补充信息（已有竞品链接、目标用户等）

提交后：
- 创建 project（status: collecting）
- 入队采集队列（从用户提供的链接或自动搜索相关话题）
- 前端轮询等待分析完成或 WebSocket 通知

- [ ] **Commit**

### 任务 10：部署配置

**文件：**
- 创建：`server/Dockerfile`
- 创建：`server/.env.example`
- 创建：`client/.env.example`
- 修改：根目录 `vercel.json`

- [ ] **配置部署**

后端：
- Dockerfile 构建 NestJS 应用
- 环境变量：MongoDB URI、Redis URI、DeepSeek API Key
- 部署到 2C4G 服务器（docker-compose 启动）

前端：
- 在现有 `vercel.json` 中为前端客户端添加配置
- 或单独部署到 Vercel，API 地址通过环境变量配置

- [ ] **Commit**

---

## 快速自检

- [x] 规格覆盖：仪表盘 + 项目详情 + 项目管理 + 采集 + 分析
- [x] 类型一致：跨任务引用的 API 端点、模型字段一致
- [x] 无占位符：所有文件路径、测试场景、实现要点具体明确
